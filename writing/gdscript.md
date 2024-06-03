# GDScript Principles

I've completed two jams in recent memory and have a pile of non-gamedev engineering experience so you really shouldn't trust me. That said after a couple of dances with Godot here's my thoughts on writing sustainable GDScript games.

## 1. Type everything, Type early
With a great type system if you can't strictly type your code it's probably doing too much at once. Unfortunately we're in GDScript and we have a mid type system but it still gets the job done.

Typing is basically a form of compiler-enforced documentation that doesn't drift. It's a useful tool in a project that you're doing solo but its usefulness grows non-linearly as the codebase gets larger than what conveniently fits in your head or the team size expands.

**When to type**  
So if typing is most useful as a project goes it's reasonable to argue "iterate quickly, type later."

Ask 8 engineers and you'll get a dozen opinions. My take is that you mostly risk future pain for (at the very best) a marginal velocity buff. Add to this that typing is an enforced contract and the GDS type system is somewhat limited it's plausible to have a perfectly reasonable untyped method that doesn't have a clean parallel as a typed solution so making a late-stage change to add typing becomes a more onerous (and dangerous) process as it introduces potentially non-trivial refactoring.

Type everything and do it from day 1.

## 2. `$<NodeName>` is empty calories

GDScript makes it extremely simple to grab nodes. Language designers care so much about this that there is a special bit of syntactic sugar: `$`. When you make something easy you're inviting it's usage, right?

It's a trap.

`$` / `get_node` are both untyped accessors that rely on scene hierarchy at a snapshot in time. It makes no assurances about what the future will be, it doesn't (currently) refactor itself as the scene tree evolves, it may break only during manual testing at run-time and potentially in surprising ways (or, worse, not until post-release).

That said for all practical purposes you're going to need to use it. To do so safely I suggest only make the call once and cache the result in an instance variable. Alternatively hide it behind an accessor method. The goal here is to limit the blast radius when you change your scene tree: for any refactor you have _one_ place you care about the tree structure not N references scattered around that you're relying on Search & Replace to fix up.

## 3. Avoid invasions of privacy
Godot leans hard on signals as a way to enable scene connectivity without introducing strong coupling. On the other hand GDScript doesn't offer visibility modifiers so there is no structural reason that you can't go around messing with internal state and, often, that's exactly how you're intended to do things (how often do we just set `Visibile`, `Position`, etc)

But that's not good! It's super reasonable for there to be some state that shouldn't be set unless the code being modified is the originator of the call/change. Also reaching into another object and fucking with attributes or methods is a quick path to tight coupling: so don't do it. The question then is how do we know what's an acceptable attribute/method to use?

Python has a convention that private variables are prefaced with `__`. It's not enforced by anything except good will but it mostly works. Golang uses case, e.g., `privateMethod` and `PublicMethod`.

So pick a convention as a proxy for private visibility and strictly adhere either by linting, code review, or an iron will. If you can't do what you want without touching private data consider making it public! If you can't safely or convincingly do that it's probably time for a refactor.

**Corollary**  
If you need access to a child node of a child node write an accessor for that and make it a public method. This is significantly more resistant to node refactor damage/code rot and has the benefit of not increasing coupling in most cases.

## 4. Singletons are a (wonderful) anti-pattern; apply as needed

This is probably my least firm conviction and has a clear path to bad behavior but: Use singletons!

Look, you're making a game not building medical software. Singletons are functionally global variables and come with all the downsides of that. However they can ales help you cut out just massive amounts of plumbing complexity and even reduce dependence on specific node/scene structure.

My guidelines are generally:

1. A singleton really should be the only one you'll need
2. All interactions should go through a strictly enforced abstraction boundary/interface
3. When possible make the logical complexity of the singleton hidden in its children and let the singleton act mostly as a shortcut to accessing those children
4. When designing singletons think about testability -- you will need some way to stub/replace the instance with a test-safe version _or_ you'll need to be resilient to your instance being not available

Doing this has (thus far) allowed me the benefits, sidestepped many of the pitfalls, and resulted in much simpler codebases. My most common use cases is having a `HUD` scene that is always available and maintains further references to things of which there should only be one of in my game, e.g., `CurrentLevel`, `DialogueUI`, `AtlasLoader`, `Configs`, etc.

## 5. Don't repeat yourself: Enumerate

I feel like this isn't going to be super controversial but if you have repeated uses of a value that has some meaning then just use an enum!

- A list of audio tracks identified by path/id? Enum!
- Standard set of animation you want to play? Enum!
- Item names? Enum!
- etc

## 6. Be data driven / use the right tool

Tweaking things in the editor can be a drag: embrace config files both for you collaborator and yourself.

Stuff like sprite atlases, animation sheets, item information often aren't most efficiently displayed/edited in-editor. Lean into that and build common file format that can import these things from where they _are_ best managed.

An argument can be made that this delays the iterative cycle since you can no longer just tweak and hit play and to some degree that's true. However if you load from the file system for debug builds it could also mean that you just don't need to rebuild and you can have creatives / designers making live tweaks to your levels/assets/whatever without needing to care about the build at all.

If there isn't an existing "best tool for the job" consider whether writing an add-on will be a net benefit / good use of time.

## 7. Don't invent it

Avoid [Not Invented Here](https://en.wikipedia.org/wiki/Not_invented_here) mentality. 

There will always be a trade-off between the total control of writing something yourself and using something off the "shelf" but if that thing won't be material to the success your vision IMO spend your time focusing on the things that are going to make your game unique.

Almost everything that you use off the shelf you can come back and replace with a custom version later if you really need to and it's probable that by this point you will have a better undestanding of you needs. This will result in a more specialized design less prone to over engineering for imagined use cases.

## 8. To be continued. I'm tired of writing now

:wave:
