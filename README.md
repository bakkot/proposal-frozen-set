# Frozen sets

JavaScript has sets and objects. It has a convenient way of getting [frozen objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze). It has no convenient way to get frozen sets.

Possibly it should! Let's discuss what that might look like in the issues on this repo.

This is _not_ a formal proposal before TC39 unless and until I or another committee member present it, which is contingent on having both interest and time. Currently it is just a place for me and other interested people to kick around ideas.


## The problem

Sets store all their data in private fields only accessible by the built-in set methods. This means that `Object.freeze` on a set doesn't prevent the set from changing. Nor does overriding the mutating methods (currently `add`, `clear`, and `delete`) to throw - someone can still call `Set.prototype.clear.call(yourThing)` to clear your thing out from under you. As long as someone has a reference to a set, they can change it.

This is unfortunate for APIs which intend only to provide their consumers query access to some data, not the ability to mutate it.


## Ideas

**Note**: I strongly recommend imagining for yourself what such an API might look like before reading this section in detail. Don't let me bias your thinking.

I'm going to list a few ideas people have suggested, without getting into pros and cons yet (though I might later). I think any of these might in principle be viable, and there might be other viable (even superior!) options not yet considered. Keep in mind that it's unlikely we'll be able to cover all use cases without adding more things than we'd care to.


### `Set.freeze` or `Set.prototype.freeze`

Just mirror `Object.freeze`. This would take a set - either as an argument in the case of a static method, as in `Set.freeze(thing)`, or as a receiver in the case of a prototype method, as in `thing.freeze()` - and modify it so that it was frozen, meaning that attempting to call any of the mutating set methods would throw. Anyone else with a reference to the set would see those methods start throwing as well.


### Read-only views

Add a set method which returns a read-only view of its argument. This would presumably _not_ be a set, but would have all of the non-mutating set methods. (Alternatively it could be a set for which all of the mutating methods threw.)

The underlying set could still be changed by someone with access to it, and the view would reflect that change. Java calls such views "[unmodifiable](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)".


### A frozen set type

Add a new constructor whose instances do not provide mutating methods. It must be constructed by passing it an iterable containing everything you will ever want it to hold.


### A "finalizable" set type

Add a new constructor whose instances are initially mutable until such time as you call `instance.finalize()`, which returns a frozen set and which invalidates the original object (such that _all_ methods called on it would now throw).


### Some combination of the above

For example, add both frozen sets and read-only views, and helpers to get from each kind to the other. All three types could get `diverge`, `readOnly`, and `snapshot` methods, where 

- `diverge` returns a new set copied from the original
- `readOnly` returns a read-only view
- `snapshot` returns a frozen set copied from the original


### True immutable data structures

Add a new constructor whose instances are like sets, including the presence of mutating methods, except that these methods return a new object reflecting the change without changing the original.


### `rekey`

There's a [proposal](https://github.com/bmeck/proposal-richer-keys/tree/master/collection-rekey) to allow sets to specify how to interpret an object as a key in the set. If this allowed distinguishing between "use this object to query" and "use this object to mutate", which it is not currently proposed to, it would allow you to build a "freeze switch" which would cause the set to start rejecting attempts to mutate it (though it is not obvious how this could be used to prevent `.clear`).


## Notes

- This discussion can be generalized to apply to maps as well as sets. It is less clear if it can be generalized to TypedArrays or other data types.
- The subject of this discussion is "shallow" freezing - that is, if the set contains a mutable object, that object will remain mutable (just as with `Object.freeze`). "Deep" freezing is a much more complex topic which is probably not in scope.
