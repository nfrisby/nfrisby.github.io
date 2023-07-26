---
layout: post
title: GHC tags thoughts 
---

My repo at <https://github.com/nfrisby/frisby-global-tags-cabal-ghc> contains a custom tool for generating tags from any Haskell project built using Cabal and GHC.

[Haskell Language Server](https://github.com/haskell/haskell-language-server) is an awesome undertaking.
For now, though, it's much too heavyweight for my taste.
My alternative tagging tool is lightweight in several ways.

- It's a standalone exe.
- It has minimal dependencies.
- The code doesn't even have to type-check, let alone build.
  It will succeed as long as `cabal config` passes and GHC can parse the code.
- It doesn't actually have a UX.
  Instead, it emits a tags file in the same format as [GNU Global](https://www.gnu.org/software/global/), and so the Emacs mode for that works.
  (Err, mostly; I have hacked my `.el` file just a little.
  I don't remember what I changed, but I know it wasn't drastic.
  And since I know almost nothing about elisp and its ecosystem, what I did could not have been sophisticated…)

The are four main ideas that drove the design.

- Work if the code is syntactically valid, even if it won't type check.
- Let Cabal determine which files to index.
- Use some clever "virtual" tags, so that there are useful and predictably-named tags for Haskell declarations that don't include names, such as instance declarations.
  Two of these standout as the most useful for me---searches that are very difficult to emulate with just `grep`.
    - `instance,C,A,B` is the tag name for an instance `C (A B)`, or `C A B`, or `C (A x) B`, and so on; the first name is the class name and the rest are the type constructors that occur in the instance's arguments.
      This syntax also works for type and data families (whether associated or not).
    - `upd,foo` is the tag of any record expression or record update expression that sets the `foo` field.
- Leverage some pre-existing UX.
  For example, the Emacs mode for GNU Global handles wildcards.
  Thus I can very easily search for all instance of the class `Bar` that involve the type constructor `Quux`!

There are a few problems/future work/caveats.

- The tool's source directly depends on the GHC API, which is notoriously brittle.
  So _someone_ will have to patch this tool for each GHC release.
- The tool's design directly depends on `cabal config` emitting the `plan.json` file.
  It reads that file to locate the source files that would be involved in the hypothetical subsequent `cabal build`; those are what it creates tags for.
  Again, I don't recall the details right now, but I once read a Cabal proposal to kill that file without replacing it 😰.
- It's a prototype; most of the code is very boilerplate-y, but it's not particularly well architected or commented.
    - In particular, one ugly chunk is a stop-gap measure for incrementality.
      It's been good enough for my non-trivial codebase, but I imagine something better could be achieved without too much more careful thinking.
- The resulting tags files are not composable, so it only lets you navigate within the local packages of your `cabal.project`.
  At one point, I had hope it could also tag `source-repository-package` stanzas, but then my team stopped using them.
  So that's unfinished.
    - I think the lightweight option is to have a means of merging the emitted GNU Global tags files---that seems imminently doable and nicely compositional.
    - Ideally, the ready-to-compose files would be bundled with the upstream releases.
    - Maybe an alternative tags format---one that can easily express the virtual tags like `instance,…` and `upd,…`---would be more amenable to such composition and distribution.

OK, that's all the thoughts I wanted to record.
If you're inspired please feel free to reach out to collaborate.
