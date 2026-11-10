<!-- The author's manifesto, verbatim from his gist: https://gist.github.com/82dfa243bab78516bee4424096516c94 -->

# Rhizome

### What software could be, if we fixed the thing we got wrong in 1965

---

This is long, because it is complete. It assumes you are clever and curious. It does not assume you write software. Every technical idea gets introduced properly, at the moment it becomes necessary, with an analogy to something you already know.

You will not need to remember a vocabulary list. Words are introduced where they earn their place.

**If you want to see it in motion before the explanation, read Part Four first.** It is one person building one real thing, start to finish. Shorter walkthroughs are threaded through the rest, each one arriving just after the idea it demonstrates.

---

# Part One — The problem

## 1. The ritual

Open any piece of open-source software on the internet. Scroll down to the instructions. You will find something like this:

> **Requirements:** Python 3.9 or later, libssl-dev, ffmpeg version 4 (not 5), CUDA 11.8, Node 18 (not 20 — it breaks). On Ubuntu run this command. On a Mac use Homebrew, but you'll need to create a symbolic link. On Windows, install Linux first. If the build fails, delete the folder called `node_modules` and try again.

Millions of these exist. Every working programmer has read thousands and written dozens.

And collectively, without anyone ever proposing it or voting on it, we have accepted that **assembling working software is a job for the human being at the end of the chain.**

Sit with how strange that is.

The machine knows what it needs. When the original author built this thing, it worked — on a real computer, with a real, specific, complete set of ingredients. Every piece of information required to reproduce it existed, exactly, at that moment.

And we throw it away. All of it. Then we write a paragraph of English prose asking the next person to reconstruct it from memory and guesswork.

That is the state of the art. Today. In an industry that has landed spacecraft.

---

## 2. What it costs you

Not in the abstract. In hours of human life:

**The afternoon lost to a conflict that wasn't real.** Two parts of your program each need a third part. One wants version 2.1, the other wants 2.6. Everything stops. You spend hours. And it usually turns out that neither part *touched* anything that differed between those versions.

**The Dockerfile.** An entire industry-standard practice whose primary purpose is to freeze a whole operating system in amber, because we could not otherwise guarantee two computers would agree on what a basic component means. We ship a copy of the world with every program, because we couldn't ship a description of it.

**The onboarding day.** Or week. A new engineer's first days spent not learning what the software does, but negotiating with the machinery required to build it.

**The rewrite that didn't need to happen.** The library that was *almost* right — it did exactly the thing, but for the wrong kind of processor, or in the wrong language. So you wrote it again. Someone had already solved your problem, and you rewrote it anyway, because there was no way to borrow the *idea* without adopting the entire cathedral built around it.

**The dependency nobody dares touch.** Pinned to an ancient version with a comment reading `# DO NOT BUMP` and no explanation, because whoever knew why has left.

Multiply by every developer alive. Multiply by every working day. Multiply by thirty years.

This is not inefficiency at the margins. **A substantial fraction of all engineering effort humanity has ever spent on software has gone into the accidental problem of making the pieces fit** — rather than into the pieces.

And almost none of it was necessary.

---

## 3. What it costs everyone

The human cost above is the one programmers feel. There is a second cost, paid by everybody, in electricity and hardware and network capacity — and it is enormous, measurable, and almost entirely avoidable.

Start with the machine you are reading this on.

### Count the browsers

Several of the applications you use daily are not really applications. They are entire web browsers with one website permanently loaded — Slack, Discord, Teams, Spotify, Notion, Figma, Postman, and the editor a great many programmers work in all the day long.

Each of them contains **its own private copy of a complete browser engine.** Not a shared one. Its own.

An empty application built this way — one that does nothing but show a blank window — is typically **over a hundred megabytes** before you write a line of your own code, because it ships a browser and a JavaScript runtime inside itself.

So the arithmetic, on a real machine, for one component:

> Five such applications installed. Each carrying roughly 150 MB of browser engine. That is **750 MB where 150 MB would do.**
>
> **Exactly 80% of it is redundant.** Not an estimate — arithmetic, for that component, on that machine.

And it is worse in memory than on disk, because those engines are not just stored five times, they are *running* five times. Each spawns its own cluster of processes. It is entirely normal for a chat application, doing nothing, to hold several hundred megabytes of RAM — most of it a browser you already have three other copies of, running.

### Count the rest

**The dependency folder.** A single modern web project routinely pulls in thousands of separate packages, occupying hundreds of megabytes to well over a gigabyte. Have five such projects on one machine and you have five copies of most of them — the same versions, byte for byte, in five places, because each project keeps its own.

**The build folder.** A Rust project's compiled output is commonly several gigabytes. Every project on your machine compiles the same shared components again, separately, into its own folder. Your computer has built the identical library a dozen times and kept a dozen copies of the result.

**The containers.** A standard base image for a common language runtime is around a gigabyte. Every service ships one. A company running forty small services is storing and moving forty copies of an operating system to run forty programs that might each be a few megabytes of actual logic.

**The phone in your pocket.** Applications each bundle their own copies of common frameworks, for the same reason: the platform cannot guarantee what will be present, so everyone brings their own.

### Now multiply it across the network

None of this stays still. It moves, constantly, in enormous volume.

**Package registries serve billions of downloads every week.** The overwhelming majority are the same tarballs, sent again, to machines that will store them alongside near-identical copies they already have.

**Container registries serve billions of image pulls every week**, each one dragging along a full operating system that is very nearly identical to the one in the pull before it.

**Every automated build starts over.** Test runs, on every change, on every branch, across every company on earth — each fetching dependencies again and compiling from something close to scratch. Caching exists, but it is keyed by whole-project fingerprints, so a single trivial change invalidates the lot and it all downloads and rebuilds again.

That traffic is real. It occupies real transatlantic capacity, is stored on real disks in real buildings, and is served from content delivery networks whose entire business is keeping copies of it physically near you — copies of copies of things that were mostly identical to begin with.

Data centres consume on the order of one and a half percent of the world's electricity, and that share is rising steeply. Nobody claims all of it is waste. But a meaningful slice of it is machines storing, moving, decompressing and rebuilding bytes that are **already, exactly, elsewhere.**

### And a security cost, which is the same cost

Every duplicated copy is also a place a flaw can hide. When a serious vulnerability is announced, the world cannot answer the simple question *where are the copies?* — because they were bundled, renamed, repackaged and re-bundled until nobody had an inventory. Remediation takes months or years, and the majority of affected software is never patched at all.

That is the identical problem wearing a different hat: **you cannot find copies of a thing when you have no way to tell whether two things are the same.**

### Why nobody can tell you the real number

Here is the part that should genuinely bother you.

**There is no reliable industry-wide figure for how much of the world's stored software is redundant.** Not because nobody cares — because it is not measurable with the tools we have. To measure it you would need to know what is actually inside every artifact, and that is precisely the information the current architecture destroys at build time.

Where redundancy *has* been measured — in container registries, in storage systems that deduplicate, in content-addressed stores — the answer is consistently *very high*. But the global figure does not exist.

**Our inability to measure the waste is the same disease as the waste.** Both come from having no way to say "this thing and that thing are the same thing."

### The worst part: we chose this deliberately

Shared libraries were invented to solve exactly this. One copy on disk, one copy in memory, used by everything that needs it. It worked, and the savings were the entire point.

Then it fell apart. Sharing required everyone to agree on *which version* of the shared thing was installed — and since things were identified by name, two programs needing genuinely different versions of the same name could not coexist. The result was a decade of failures so notorious they got their own name: *DLL hell* on one platform, *dependency hell* on the others.

So the industry gave up and went the other way. Bundle everything. Ship your own copy of the world. Put it in a container. Statically link it. **Duplication was consciously adopted as the lesser evil** — and it was, given the alternative.

That is the honest history. The waste is not carelessness. It is the price we agreed to pay to escape something worse.

**But we only had to make that trade because identity was a name.** Give things an identity computed from their contents, and sharing no longer requires anyone to agree on anything. Two programs needing genuinely different versions get genuinely different things, with no conflict, while everything they truly have in common is stored exactly once, automatically, without coordination, forever.

The trade stops being necessary. And so, then, does the waste.

---

## 4. The culture that grew up around it

This part is uncomfortable, so let's be direct about it.

When a system is broken in a way that cannot be fixed, the people inside it stop treating the brokenness as a problem, and start treating **mastery of the brokenness as a qualification.**

Knowing which obscure command fixes the build error becomes a mark of seniority. Reading a four-hundred-line error message and finding the one relevant line becomes a party trick performed for juniors. Knowing which four invisible settings must be configured first becomes tribal knowledge — the sort of thing you are gently mocked for not having.

And the people who bounce off all this get a label: *non-technical. Not really a programmer. Not very bright.*

They are not stupid. They looked at a system demanding they memorise a thousand arbitrary facts that encode nothing, teach nothing, and expire in two years — and concluded this was a poor use of a human mind.

**They were right.** And we punished them for being right.

We turned accidental complexity into an initiation rite. We filtered the field not by capacity to think but by tolerance for pointless friction. And we lost — permanently — an enormous amount of talent that would have been brilliant at the actual work, because the entrance was a wall of undocumented incantations guarded by people who enjoyed knowing them.

The README that says "install these fourteen things first" is not documentation. It is a toll booth. A culture that treats operating the toll booth as an achievement has confused *surviving a bad system* with *skill*.

We can stop. That is what this document is about. And we can stop now specifically, because the tools required arrived while everyone was looking elsewhere.

---

## 5. The actual mistake

All of the above traces to three decisions. All were reasonable when made. All are now load-bearing and rotten.

### Mistake one: things are identified by name

A piece of software is referred to by a label a human chose. Rename it, move it, reorganise the folder it lives in — and every reference to it, everywhere in the world, breaks.

The patch is a nickname table mapping old names to new ones. Those nicknames then *expire* after a grace period. So the mechanism intended to provide stability has a shelf life.

> **The problem in one image.** Imagine if your identity were your address. Move house and you become, legally, a different person — and everyone who knew you must be individually notified or they lose you forever. We would call that an absurd way to run a society. It is how we run software.

### Mistake two: versions describe whole components

"This needs version 2.1, that needs 2.6" is a claim about entire components. But the truth lives at a much finer grain: *which specific functions does each part actually use, and which specific functions actually changed?*

Because we can only see the coarse version numbers, we duplicate, or we fail, or we spend the afternoon.

> **The problem in one image.** Two recipes both call for flour. One says brand A, one says brand B. Rather than checking whether the difference between the brands affects either recipe — it almost never does — the kitchen refuses to proceed until you buy two separate bags and label them.

### Mistake three: boundaries are treated as walls

A package, a library, a program. Each a sealed box whose walls are structural. Changing anything means negotiating with every wall. You can *use* what's inside but not *see* it, and certainly not reshape it.

Over time the shape stops being a convenience and becomes the thing itself. Everyone forgets it was ever a choice.

### And underneath all three: isolation is the default

Here is the inversion so old we stopped seeing it.

**Everything is sealed off by default, and connecting things is work you must do.** Every program is a box. To make two of them cooperate you write glue, define a protocol, version an interface, and maintain that seam forever.

But that is not the nature of software. Software *interacts* — that is what it is for. It drives machines, moves money, connects people. Interaction is not the exception; it is the entire point. And we made it the hard path.

**It should be the other way round.** Connection should be the default, because things connect. Isolation should be the deliberate exception, declared where it is genuinely needed.

### Where all this came from

Nobody argued for any of it. It was inherited.

Early computers read one file at a time, because that is what the tape drives allowed. So the file became the unit of compilation. Then languages invented *modules* to name those units, *packages* to bundle the modules, and *imports* to wire them together.

The entire structure — files, modules, packages, imports, visibility rules — is downstream of a hardware limitation from the 1960s.

It was never designed. It calcified.

---

# Part Two — The idea

## 6. Identify things by what they are

Instead of identifying a piece of software by a name someone chose, identify it by a **fingerprint computed from its own contents.**

There is a standard tool for this. A *hash function* takes any input and produces a short string of characters. The same input always gives the same string. Any change to the input gives a completely different one. It is a reliable, compact summary of "exactly this and nothing else."

Use that fingerprint as the thing's identity.

Now rename it, move it, reorganise everything around it — the fingerprint is unchanged, because it never depended on the name.

> **Analogy.** A library that shelves books by their exact contents rather than their titles. Two identical copies land in the same slot automatically, because they *are* the same book. Retitle one and nothing moves, because the shelf position never depended on the title.

**Names don't disappear** — nobody wants to type a fingerprint. They become *labels pointing at* fingerprints. Nicknames. And that is the whole change: a nickname can be changed, can be personal, can be disputed, without the thing itself being touched.

### The version worth stealing: fingerprints that include their ingredients

Take it one step further. Compute each thing's fingerprint from its own contents **plus the fingerprints of everything it depends on.** Since those were computed the same way, this ripples all the way down.

Now a fingerprint answers a much better question. Not "is this piece the same?" but **"is this piece, and every single thing underneath it, exactly the same?"**

That second question is the one that actually matters, and answering it normally requires an expensive investigation.

> **Analogy.** A recipe whose identity includes the identity of every ingredient. Change the flour supplier, and the cake's fingerprint changes automatically, without anyone checking. You never have to ask "did something underneath me change?" The number already told you.

This technique is called a *Merkle tree*, after Ralph Merkle who described it in 1979. It is what makes your git history tamper-evident and what lets your computer verify a download.

**One wrinkle.** Occasionally two things refer to each other in a loop. Neither can be fingerprinted first, since each needs the other's answer. The fix is to treat the mutually-dependent group as one unit and fingerprint it together.

> **Analogy.** Two people who each list the other as their emergency contact. You cannot describe one without the other, so you treat the pair as a single entry.

### The trick that makes it much stronger

Compute the fingerprint from the *structure* of the code, not its text. Then formatting, spacing and even the names of internal variables can be excluded.

Which means: two people who make the same change, independently, in different styles, with differently-named variables — **produce the identical fingerprint.** Same thing. Automatically recognised as one, stored once.

> **Analogy.** Recognising that two people described the same route, one saying "left at the church" and the other "left at the church on the corner." Same route. The words differed; the structure didn't.

This is called *alpha-equivalence*, and it comes from mathematical logic, where it has been standard for decades.

The honest limit: two solutions that reach the same answer by genuinely different methods will never be recognised as identical. That is not a bug, it is a proven mathematical impossibility. Same-shape-different-wording is the achievable win, and it catches most real duplicated work.

---

### ▸ Walkthrough — two strangers, one fix

**Tomás** hits a bug in a date-handling function and fixes it. He names his loop variable `attempt`, indents with tabs, leaves a blank line before the return.

**Wen**, on the other side of the world, three weeks later, hits the same bug and makes the same fix. She names the variable `tries`, uses spaces, no blank line.

Neither knows the other exists.

**They produce the identical fingerprint.** The formatting was never part of it. The internal names were never part of it. Only the structure was — and the structure is the same.

So there is one fixed version in the world, not two. Anyone who arrives later and makes the same fix lands on it too, automatically.

> Today this is two forks, drifting apart, each maintained by someone who doesn't know about the other, both eventually rediscovering each other's edge cases the hard way.

---

## 7. There are no files

If everything is identified by fingerprint and every reference is a fingerprint, then something surprising follows.

**Import statements have no job.** An import exists to tell the computer *where to go looking for a name*. Remove name-based lookup and the entire concept evaporates. A reference already points directly at the thing.

Then **files have no job either.** A file is a box that groups some code and separates it from other code. But the grouping is now visible in the fingerprints, and the separation was never real.

So there are no files. Not one big file — **no files at all.** There is a graph: definitions, and the links between them.

### But wait — how do you look at it?

Through **views**. You ask the system to show you a slice: everything related to this thing, everything that calls it, everything it calls, the sequence in which things happen. You choose the slicing, and you switch between slicings.

Crucially this is *not* drawing programs with boxes and arrows. That has been tried repeatedly and works badly. You type ordinary code, in an ordinary text box. What changes is **navigation**: you move through relationships instead of through folders.

> **Analogy.** The difference between a filing cabinet and a good database. The cabinet forces one arrangement — alphabetical by surname, say — and finding everyone in one city means opening every drawer. The database has no primary arrangement; you ask for what you want, and get exactly that view. Neither view is "the real one."

This is not speculative. Smalltalk worked this way in the 1970s — you browsed classes and methods, never files, and edited one method at a time in a normal text pane. It ran serious production systems for decades and still does.

### What you gain

In files, a relationship can hide. A function five files away calls a function three modules over, and nothing in front of you says so. You have to already suspect, in order to go looking.

In a view computed from real relationships, **it is in front of you by default.**

The information is technically available today — any modern development tool can show it on request. The change is from *available on request* to *present without asking*. Something you must ask for gets consulted when you already suspect you need it, which is exactly when you least need reminding.

> **Analogy.** The difference between a map you can consult and a map on the wall. The information is the same. What you notice is completely different.

### Two things you must not lose

**Where things are.** People navigate partly by spatial memory. A view that rearranges itself between sessions quietly destroys that, and people feel lost without being able to say why. So layout is remembered — but as *your* layout, personal to you, never affecting anyone else.

**The order the author intended.** A written file carries a sequence: the important thing first, the helpers below. That ordering is real communication. So authors can assign a display priority — a suggested reading order — that any reader may follow or override.

---

## 8. Take the behaviour, not the box

You want one function. Today you get the library it lives in — which drags its own dependencies, which drag theirs.

Instead: start from what you actually want, follow every reference, follow the references of those, and stop when nothing new appears. That set — exactly what is needed, nothing more — is called the **closure**.

> **Analogy.** Packing by following the real chain of need. You pack the charger because you packed the laptop. You don't pack the whole house — but you also don't forget the charger.

This has been impossible until now for a mundane reason: computing the true minimal set is expensive, and approximating it with "just take the whole package" was cheaper. Cheap computation flipped that.

### Grafting

Taking a piece of someone else's software into yours is called **grafting**, after the gardening technique: cut a branch from one tree, attach it to another, it grows as part of the new tree.

That is precisely the operation. Take the piece. Attach it. Keep working. And the graft remembers where it came from, so improvements can go home.

This is also the answer to "should we rewrite everything?" No. **Wrap what exists, replace it gradually underneath.** This has already happened repeatedly in real life — several major security and compression libraries were first wrapped and later replaced entirely, with everyone using them noticing nothing at all.

---

### ▸ Walkthrough — Priya needs one function, not the cathedral around it

**Priya is building a tool** and needs one specific date-parsing behaviour. It exists, perfectly, inside a large library.

**Today** she adds the library. It pulls in three hundred and forty other components. Her build time triples, her binary gains forty megabytes, and she has taken on a maintenance relationship with an entire ecosystem — plus a security surface she cannot audit — in order to use one function. Or she reimplements it herself: three days, subtly wrong at the edges, and now the same bug exists in two places in the world.

**Here she grafts it.** She points at the function. The system follows what it genuinely needs — eleven other definitions, as it turns out — and that is what arrives. Not the library. Not the library's dependencies. Eleven definitions, and she can read every one of them.

**Then the second half of her problem.** The function only works on one kind of processor and she needs another. Today that means either persuading a stranger to accept a substantial change, or forking and maintaining it forever.

Here she writes an additional layer with a condition on it. It activates only for the processor it is for. On every other machine it is never fetched and never built — as far as anyone else is concerned it does not exist.

Which means the original author can accept her contribution **without checking whether it breaks everyone else**, because the structure makes that impossible. It can only add.

> **What this changes about open source.** Maintainers are slow and cautious not because they are unhelpful, but because every change is a risk they personally absorb on behalf of thousands of strangers. Remove the risk and the caution goes with it.

**What is still hard:** grafting across genuinely different language runtimes — pulling something out of a language that carries its own garbage collector and scheduler — is a real bridge with real cost. Within one language it is nearly free. Across runtimes it is engineering.

---

## 9. Changing something is not editing it

Here is where it becomes properly interesting.

You find a function deep inside something you're using. You change one line. It feels like an edit.

**It isn't.** Because identity is contents, a changed thing is a *different* thing with a different fingerprint. The original still exists, unchanged, and everyone using it is entirely unaffected. Forever.

And a consequence follows automatically: your changed function has a new fingerprint, so whatever called it must now point at the new one — which changes *its* fingerprint, and so on up the chain to the top of your project.

That sounds expensive. It is not. Each link in that chain is a tiny record saying *"identical to the previous version, except this one reference now points somewhere else."* No code is copied. Every untouched neighbour is shared.

And it is bounded by *your* project, not by the world. If that function has five hundred users and your project touches three of them, three chains re-point. The other four hundred and ninety-seven never learn this happened.

> **Analogy.** A change of address that ripples only through the people who actually needed to know, and reaches nobody else — because nobody else was in the chain.

### What this deletes

**Version numbers.** Entirely. No ranges, no compatibility matrices, no solver, no conflicts. You hold exact fingerprints. Nothing can move underneath you, because there is no "underneath you" that anyone else controls.

**Dependencies, as commonly understood.** You are not declaring "I need something roughly like this." You needed a change in something, you made it, and now you hold your own version of the entire chain — deduplicated, so it costs nothing.

**Merge conflicts.** Two people editing different definitions cannot conflict, ever, because there is no shared page for their edits to collide inside. Textual merge conflicts turn out to be an artefact of storing a graph in files.

What remains is a **succession link**: a note that this thing was derived from that thing. Not a number — just lineage. Without it you'd have an ocean of unrelated fingerprints and no way to ask "has anyone published a fixed version of what I'm running?"

### Frozen — and the honest cost

Your software is frozen exactly as you left it. Nothing updates unless you say so. Nothing breaks on a Tuesday because someone changed something.

**But frozen also means unpatched.** A flaw found deep in your closure does not reach you until you act.

That is a deliberate trade, and the machinery to make it painless already exists in this design:

- Because every reference is a fingerprint, "who is affected by this flaw?" is a **lookup** rather than an investigation. This is the question that costs the industry *years* after every major security incident — hunting for copies buried in software nobody has an inventory of.
- The system can classify the fix automatically: *drop-in* / *needs data conversion* / *needs a rebuild*. So the alert arrives carrying its own risk assessment, computed rather than claimed.
- And you choose, per item, between three settings: **pin** (never move), **follow** (take fixes automatically), or **defer** (whatever this person I trust approves).

> **Analogy.** The difference between a landlord who lets themselves in and redecorates while you're at work, and a landlord who texts to say the boiler model you have has a known fault, here's the fix, and it's a ten-minute job. Nothing happens without you. But you are never left not knowing.

### On trust: keep disagreement visible

If two people you delegate to approve different fixes, the system should **show you both** and who stands behind each — not pick one.

The disagreement is the useful signal. Two informed people examined this and reached different conclusions; that is worth seeing.

Treat it as an error requiring resolution and everyone will quietly converge on whichever authority resolves fastest — and you have rebuilt a central gatekeeper with extra steps.

---

### ▸ Walkthrough — Marco has a bug in someone else's code

**Marco's product is broken** by a bug three levels down inside something he depends on.

**Today his options are all bad.** Wait for the maintainer — weeks, months, possibly never. Fork the whole thing and vendor a copy, taking on permanent maintenance and drifting from upstream daily. Or write a workaround at his own level and leave a comment apologising to whoever reads it next.

**Here he navigates to the function** — not by guessing which file it lives in, but by following the actual chain of what calls what, from his own code down to the thing misbehaving. On the way he can see everything else that touches it.

He changes one line.

**It looks like an edit. It is a divergence.** The original is untouched and everyone else using it is unaffected, permanently. Marco now holds his own version of that function and of the short chain of things that referred to it, up to his product. Each link is a small record saying *"same as before, except this one reference points elsewhere."* The other four hundred and ninety-seven users of that function never learn this happened.

His inspector says it plainly: **you diverged four things; the substance you changed is one line.**

**Then the good part.** He offers it upstream. Because the system knows exactly what changed — structurally, not as a text difference — the maintainer sees precisely that one line, with no noise from formatting or naming.

If they take it, his version and theirs become the same fingerprint and merge into one thing. If they don't, Marco keeps his at **zero ongoing cost**, because nothing underneath him will ever move without his say-so.

> Today, "I fixed it in my fork" is the start of a long unhappy relationship. Here it is a stable end state that costs nothing to remain in.

---

## 10. No configuration files, no settings, no environment variables

Software today is configured through at least four separate mechanisms: command-line arguments, environment variables, configuration files, and build-time flags.

All four exist for one reason: **the finished program is a sealed box, so values have to be smuggled in from outside.**

Once a value is simply another item in the graph, and configuring means pointing a reference somewhere else, that entire category dissolves. There is nothing to smuggle. You change the value the same way you change anything.

Two clarifications keep this honest.

**Some values are structure, not settings.** The `2` in a doubling function is not a knob. So values can be *marked as exposed* — advertised as intended settings, with the author's choice as the default. Unmarked values remain changeable; they're just not advertised.

**Some values cannot exist until the program runs** — the current time, an incoming request, a network port that happened to be free. Those genuinely arrive at runtime and always will.

### Secrets, handled properly

Passwords and keys must **never** be in the graph. Two independent reasons:

1. **You cannot delete from a fingerprint-addressed store.** Others may reference it. A leaked secret is leaked permanently.
2. **Fingerprints of short values can be reversed** by simply trying every possibility. Publishing the fingerprint of a password is close to publishing the password.

So the graph holds a **handle** — a reference meaning "the value behind this," resolved outside the graph when the program runs. The real value lives in a private compartment on your machine, or is encrypted specifically for the machines you choose.

This turns out to be better than today's arrangement in three ways:

- **Rotation is free.** Swap the value behind the handle; nothing in the graph changes, nothing rebuilds. (Today, rotating a secret often means rebuilding and redeploying — which is exactly why people don't rotate.)
- **Handles can be weakened.** Hand someone read-only access, or access that expires in an hour, without them being able to widen it.
- **Revocation is instant.** Invalidate the handle and everything holding it loses access at once.

This is an old, well-studied idea called *capability-based security*: a thing that both names a resource and grants access to it, with no ambient permissions floating around.

> **Analogy.** A hotel keycard rather than a master key with a list of rules attached. The card *is* the permission. Cancel it and the door simply doesn't open. Nobody has to update a list.

Incidentally, this removes environment variables — which are genuinely dangerous for secrets. They are readable by other processes, silently inherited by every program your program starts, and routinely captured in crash reports.

---

### ▸ Walkthrough — Ana tunes something, and the knowledge survives

**Ana runs a media server.** She spends an afternoon working out that one buffer value should be far larger for her hardware and her workload. Throughput improves by a third.

**Today that knowledge has three possible fates.** She tells nobody and it dies with her setup. She writes a blog post that is accurate for eleven months and misleading for the next nine years. Or she posts in a forum thread, where it becomes folklore — *"set it to 64, worked for me"* — no hardware, no workload, no way to check, quietly rotting.

**Here she attaches the measurement to the value itself.** Not a star rating. A claim with its conditions: this value, this hardware, this workload, this measured result.

And because the configuration is exact, **anyone who doubts her can reproduce it.** That has never been possible before. Advice about software settings has always been fundamentally untestable — you could never be certain you were running what they were running. Here you can be, because what they were running is a fingerprint.

Folklore becomes evidence.

**What is still hard:** popular is not the same as correct-for-you. A value tuned on a sixty-four-core server is wrong on a laptop. So what is worth trusting is the *evidence and its conditions*, never the number on its own — which is exactly why the measurement carries its context rather than being a score.

---

## 11. The end of searching the internet for your own computer

Think about what you actually do today when you want to change something about a program you already have.

You do not ask the program. You ask **the internet**. You open a browser, type the name of the software and a rough description of what you want, and you land in a forum thread from four years ago where a stranger says *"set it to 60, worked for me."* No indication of what hardware they had. No indication of whether it's still true. No way to check. You try it. It doesn't work, or it does and you never learn why.

The information you needed was **on your own machine the entire time.** It was simply unreachable, because it was sealed inside a box, described in prose that lives somewhere else, and findable only by a search engine that has never seen your computer.

### Why documentation always rots

Here is the root of it, and it is the same disease as everything else in this document.

**Documentation today is a copy of the truth.** It's prose written *about* the software, stored separately from it, describing how things were at the moment someone wrote it down. Two things then drift apart at their own pace, and nothing connects them. The software changes; the prose doesn't. Nobody notices until you follow the instructions and they don't work.

Every remedy ever attempted — doc comments, generated references, wikis, "docs as code" — is an attempt to keep the copy close to the original. It helps. It cannot fix the underlying problem, because there are still two things.

**Here there is one thing.** When you look at a setting, you are not reading a description of the setting. You are looking at the setting. It cannot be out of date, in the same way your own reflection cannot be out of date.

> **Analogy.** The difference between a museum placard and the object itself. Placards get replaced, mislabelled, left describing an exhibit that was moved years ago. The object simply *is* what it is. You are no longer reading the label. You are holding the thing.

### What the search actually reaches

And what you're searching is far richer than documentation has ever been, because every exposed value carries with it:

- **its name**, and every alternative name anyone has ever given it
- **its author's default** — the value chosen by the person who built it
- **its explanation** — the doc, written in place, unable to drift
- **its lineage** — every change it has been through and who made each
- **what it affects** — everything downstream, computed exactly rather than guessed
- **and what other people actually measured**

That last one has no equivalent anywhere today, and it is the one that changes the experience most.

Right now, the collective knowledge of everyone who has ever tuned a piece of software exists as **folklore** — scattered across forums, blog posts and comment threads, unverifiable, uncontextualised, and slowly going stale. Rhizome turns it into **evidence**: a claim like *"set to 64, throughput improved 12% on this hardware with this workload"* attached to the exact value it concerns, bound to the exact configuration it was measured on, and — because the configuration is exact — **reproducible by anybody who doubts it.**

You don't have to trust the stranger. You can check.

### The five-minute path, step by step

Say you want to change a frame rate limit inside a game compatibility layer. Today this is a browsing expedition of unpredictable length. Here it's a narrowing:

1. **Search "fps".** Plain language, no need to know what the setting is really called.
2. **Narrow to what you're actually running.** Not all software everywhere — just this thing and everything beneath it.
3. **Show only the knobs.** Values the authors marked as intended settings, rather than the thousands of internal numbers that are structure rather than configuration.
4. **Read what each one does, and what other people found.** In place, current, with evidence.
5. **Change it.** Which, as established, is just pointing a reference somewhere else — instant, reversible, and affecting nobody but you.

None of these steps requires new machinery. Step 2 is the same walk that works out what a program needs. Step 3 is the *exposed* mark from the previous section. Step 4's downstream list is the same index that finds who's affected by a security flaw. Four capabilities that already had to exist, recombined into a search box.

### Two details that make or break it

**Synonyms come free, and they matter enormously.** Someone searching "fps" may need a value called `frame_limit`, or `max_frame_rate`, or `refresh_cap`. Because names are labels pointing at fingerprints, and many labels can point at one fingerprint, **an alternative name is a search term** — and anyone can add one without touching the software. Community-contributed names become community-contributed findability. This is the thing that makes plain-language search work over code written by thousands of strangers with different vocabularies.

**Ordering is the hard part, not finding.** Searching "fps" across a large graph returns thousands of matches, and the difference between a five-minute answer and a wasted hour is entirely which one is at the top.

But the ranking signals available here are extraordinary, and none of them exist today: how many programs use this value, how often people override it rather than leaving the default, whether anyone has measured it — and above all, **how close it is to what you are actually running.** A setting inside the software on your machine should outrank an identically-named setting in software you've never installed.

No search engine on earth can do that, because no search engine has your graph. This one does.

### The honest limit

This eliminates *lookup*. It does not eliminate *explanation*.

"Which setting, and what does it do?" becomes a query with a fast, current, trustworthy answer. But "*why* does frame pacing behave this way, and what am I trading off against latency?" is still someone writing an explanation well, and no architecture can conjure that.

What it can do is make the good explanation **attach to the thing it explains** — so it's found by everyone who needs it, at exactly the moment they need it, instead of sitting in a blog post that will outlive its own accuracy by a decade.

Which, honestly, is most of the value anyway. The best explanations already exist. They're just not where you're looking.

---

### ▸ Walkthrough — Sam wants the chime to stop

**Sam is a nurse.** Not a programmer, has never wanted to be. On night shifts their messaging app chimes for every message, and they want it to chime only for the ward group. The app has no such setting.

**Today this is not difficult — it is impossible.** Sam can post on a forum and get no reply. They can file a request that sits unread for three years. They can leave the app permanently silent and miss the messages that matter. The code that decides whether to play that sound is open source, sitting on a public server, and might as well be on the moon.

**Here Sam types "notification sound".**

What comes back is not six settings someone chose to expose. It is everything in that app touching notification sounds — including internal decisions the author never thought of as configurable. Each shows what it does, in place. Sam can see that one of them is the yes-or-no decision to play a sound at all.

And something else: **four other people have already changed this.** One variant is described as *per-group sound control*, is used by eleven thousand people, and carries a note about what it handles and what it doesn't. Sam takes it. Two clicks.

**Underneath:** an entry in Sam's personal name list now points somewhere else, and the chain re-pointed up to their copy of the app. Everything that didn't change is still the same shared thing it was five minutes ago. Nothing was downloaded except the few definitions that genuinely differ.

**What is still hard, honestly:** if nobody had made that variant, Sam would have to — and that means understanding the code. The distance went from *impossible* to *hard*, not to trivial.

But that is the entire difference. Impossible has no path through it at any level of effort. Hard has a path, and Sam gets to decide whether to walk it, ask someone, or wait for somebody else. Today that path does not exist.

> Note what is absent. No "download the source." No "install a development environment." No "clone the repository and learn the build system." Sam did not become a developer. Sam changed a setting that happened to sit one keystroke deeper than the settings screen goes.

---

# Part Three — How it runs

## 12. The program is a receipt

Here is a shift that sounds technical and is actually the key to everything.

**The finished program is not the valuable thing.** The graph is. The program is a *consequence* — something recomputed automatically from the source that is on your machine right now.

And because identity is exact, the system can answer "have we built this exact thing before?" instantly, and skip the work.

### Two kinds of work, two kinds of reuse

Preparing software has two halves that behave completely differently:

**Understanding it** — reading it, checking that it makes sense, confirming everything fits. This is identical regardless of what machine will eventually run it. **It can be done once, for everyone on earth.**

**Preparing it for one specific machine** — turning the understood code into instructions for a particular processor. This must be redone per kind of machine, but it's the cheaper half.

> **Analogy.** Translating a book. Working out what the author meant is done once and is the hard part. Setting the type for a paperback versus large print is redone per edition and is the easy part. Today's tools redo the entire translation for every edition, on every machine, over and over.

Split those two and the expensive universal half becomes shareable with everybody, while only the narrow machine-specific part is ever repeated.

### Compiling only what changed

Every step is recorded along with what it depended on. Change something, and only steps that actually depended on it are marked stale. Everything else stands.

> **Analogy.** A kitchen that keeps notes. *Onions chopped, in the fridge.* Next order, you don't chop again. Someone swaps the onions? Only notes mentioning onions get torn up. Yesterday's stock is still fine.

Change one line in one function, and that function is rebuilt. Nothing else.

### And the same machinery gives you live values

Because a pure calculation with exact inputs always produces the same answer, **results can be stored by fingerprint too** — exactly like compiled code.

Which means the environment can show you, sitting beside your code as you work, what things actually evaluate to. Not by running a session that accumulates hidden state — the notorious flaw in existing tools of this kind, where results depend on what you happened to run earlier and in what order — but as a stored, reproducible fact that anyone can independently verify.

One mechanism. Three features: compilation caching, configuration, and live values. No separate systems to build, document or debug.

---

## 13. Everything visible, almost nothing present

Your machine appears to contain the entire tree of all software. Almost none of it is actually there. When something is genuinely needed, it arrives at that instant. Untouched things never arrive.

This is done at the deepest level — the filesystem itself — so that *every existing tool works unchanged*, believing it is looking at perfectly ordinary files.

> **Analogy.** A streaming service. The full catalogue is on your screen; nothing is on your disk until you press play. And critically, it behaves like a local library, so nothing has to be taught about it.

This is thoroughly proven. Google, Microsoft and Meta each independently built exactly this for their own enormous codebases — and the fact that three organisations solved the same problem the same way, without coordinating, is strong evidence the solution is not arbitrary but forced by the shape of the problem.

---

### ▸ Walkthrough — Dana joins the team on Monday

**Today Dana's first day is a setup day.** Sometimes a setup week. The instructions are eighteen months old, two steps fail, and someone senior loses an afternoon helping. By Wednesday it works, mostly, and there is one thing everyone has learned to ignore. Every person who ever joins repeats this, and the knowledge required is stored nowhere except in the heads of those who already suffered it.

**Here Dana connects and pulls the project. It runs.** Not "should run" — it is, verifiably, the same thing everyone else on the team is running, down to the last fingerprint. Nothing to install, nothing to configure, no environment to reproduce. There is no *works on my machine*, because there is no my-machine version of it.

**Then the half that matters more.** She isn't handed four hundred files and told good luck. She's handed something she can ask questions of.

*What happens when a request arrives?* — the sequence, with routine parts folded away until she wants them. *What touches billing?* — exactly that, not a text search that finds the word in comments. *If I change this, what breaks?* — an exact answer rather than an educated guess.

And when she can't picture what a function does, the values are sitting beside it — real, computed, stored, not a session she has to set up and run.

> **Analogy.** The difference between being handed a city's phone book and being handed the city. Same facts. Only one lets you see how anything connects.

**What is still hard:** *why* the system was designed this way — the decisions, the trade-offs, the roads not taken — is still someone explaining it. The graph shows Dana what is. It cannot tell her what was considered and rejected.

---

## 14. One kitchen, many dishes

A single long-running process keeps all the expensive shared groundwork loaded and warm. Each program you start begins as a *copy* of that warm base rather than from nothing.

The copy is faked, cleverly: both share the same memory until one writes to it, and only then is that piece actually duplicated. So the copy is nearly free and stays nearly free.

> **Analogy.** A pizza kitchen with the dough already made and the oven already hot. Every order starts from that shared base. Each cook appears to have a full pantry, but they're sharing the shelves until someone actually changes an ingredient.

This is not novel — it is exactly how the phone in your pocket has launched apps for fifteen years.

**Each program still gets its own separate space.** This might seem to contradict everything above about removing boundaries. It doesn't. The boundaries being removed are *filing conventions* — packages, modules, files, which were only ever labels. A separate memory space is a *physical* boundary: it means one program crashing doesn't take the others with it, and one being compromised doesn't hand over your passwords.

> Sharing a toolbox is excellent. Sharing a workbench where anyone can knock over your work is not.

### And security comes free

Here is the elegant part.

The system already worked out exactly which pieces went into building your program. **That same list becomes the program's entire visible world.**

Normal sandboxing is a list of rules saying "you may not touch that" — which someone must write, maintain, and get right. Here there is no list. Things your program doesn't need aren't *forbidden*; they are **absent**. And they're absent for free, because the calculation was already done for another purpose.

> **Analogy.** A hotel keycard that doesn't merely refuse the other floors — the lift doesn't display them. And the floors it does display were derived from your booking, not from a separate security policy that someone maintains and occasionally gets wrong.

---

## 15. Changing things while they run

Calls don't go directly to their destination. They go via a small index of "where each thing currently lives." To replace something in a running program, you update its entry. Everything in flight keeps working.

> **Analogy.** A theatre where the script sits on a stand, each scene a page in a slot. To change a scene mid-performance you swap the page. The actors keep reading from the same stand and never notice.

And crucially, the system can determine *in advance* whether a given change is safe to apply live — using the very same analysis that resolves version conflicts:

- only the inside of a function changed → **swap it live**
- the shape of some stored data changed → **existing data needs converting first**
- a fundamental boundary changed → **restart required**

Today, live updating is largely try-it-and-hope. Here it is a computed answer, known beforehand.

> **Analogy.** A mechanic who knows before starting whether the job can be done with the engine running, needs it switched off, or needs the engine out. The knowing-beforehand is the entire value.

This is not exotic, either. The Linux kernel — the most safety-critical software most people run — has replaced individual functions in running systems without rebooting, in production, for years.

---

### ▸ Walkthrough — a serious flaw is found in something everyone uses

**Today this is one of the ugliest recurring events in the industry.** A flaw is announced in a widely-used component, and the world spends *months* — sometimes years — answering the question **"where are the copies?"**

Because things are vendored, bundled and repackaged, nobody has an inventory of what is inside their own software. An entire compliance industry exists to reconstruct, afterwards, information that was discarded at build time. Governments have written laws requiring companies to produce lists of what their software contains — which tells you how thoroughly the architecture lost that information. And most people running the affected software never find out at all.

**Here the question is a lookup.** Every reference is a fingerprint, so "who reaches this?" costs what any other lookup costs. Not a scan, not an estimate — the exact set.

Then the fix is classified automatically by the same machinery that decides whether a change can be applied to a running program: *inside of a function only* (drop-in), *shape of stored data changed* (needs conversion), *fundamental boundary changed* (needs a rebuild). The notice arrives carrying **its own risk rating, computed rather than claimed by whoever published it.**

Everyone affected gets it. Not just developers — **there is no separate category of person here.** Sam the nurse gets the same notice about their messaging app that its maintainer does, in the same form, with the same one-click path.

What happens next is each person's choice, made in advance, per item: **pin** (never move, but tell me), **follow** (apply fixes as they appear), or **defer** (do what this person I trust approves). Sam pins the three things they care about and defers the rest. Marco, running a production service, pins everything and reads every notice himself.

**And when two people Marco trusts disagree** — approving different fixes for the same thing — he is shown **both, and who stands behind each.** The system does not pick. The disagreement is the useful signal: two informed people looked and reached different conclusions. Resolve it automatically and everyone quietly converges on whichever authority resolves fastest, and you have rebuilt a central gatekeeper with extra steps.

**The honest part, stated plainly.** Because nothing ever changes underneath you, **nothing is ever silently fixed either.** A flaw sits in your closure until you act. That is a deliberate trade — and the entire apparatus above exists because if you are going to make that trade, the path from *a fix exists* to *I am running it* has to be one click and a computed risk rating, not an archaeology project.

And the system finds *who is affected*. It does not find *the flaw*. Somebody still has to notice the bug and write the fix. That remains human work and always will.

---

# Part Four — A whole piece of work, start to finish

The walkthroughs above each showed one idea in isolation. This is all of it at once: one person, one real project, from the first keystroke to a stranger on the other side of the world running it. It is the closest thing to a picture of what the day feels like.

---

## 16. Yusuf builds something

Yusuf wants to make a small thing: a tool that watches a folder of photographs, recognises which ones are near-duplicates, and quietly files the extras away.

### Opening

He opens a blank surface.

It looks like an empty text editor, and it is not a file. Nothing is created on disk. There is no project folder, no configuration to initialise, no template to choose, no build system to set up. There is a place to type, and it exists the moment he asks for it.

He types ordinary Rust.

**What is different is the autocomplete.** He starts writing something about image hashing and the suggestions are not drawn from a list of things he has imported — he has imported nothing, and there is nothing to import. They are drawn from everything that exists. Every perceptual-hashing function anyone has ever written, each with its explanation attached, each showing how many other people use it and what they measured.

He picks one. There is no install step. He referenced it; it is already reachable.

### As he types

Beside his code, values appear.

Not from a session he set up and ran — from stored, reproducible facts. He writes a function that turns an image into a comparable fingerprint, and next to it sits the actual fingerprint of the actual test photograph, computed and remembered. He changes a threshold; the number beside it changes. He changes it back; the old number returns instantly, because it was never thrown away.

> **What this replaces.** The write-build-run-look-change loop, performed thousands of times a day by every programmer alive, each iteration costing anywhere from two seconds to two minutes of dead waiting. Here the answer is beside the question, and it is already correct.

### Loosely, at first

Yusuf is not being careful about efficiency, and the system is not making him be.

He is working at the loose end of the dial: copying data freely, not thinking about who owns what, letting errors be vague. It reads almost like Python. It is not a different language or a simplified subset — it is the same code, expressed loosely. It uses more memory and offers fewer guarantees, and for a tool sorting his holiday photographs, that is entirely correct.

**He gets something working on the first evening.** That has never been the Rust experience.

### Going deeper

The near-duplicate detection is wrong on a particular kind of photograph. He needs to see why.

He navigates — not through folders, but along the actual chain of what calls what, from his own code down into the hashing function he borrowed, and further down into what that uses. Everything related is in front of him. Nothing is hidden at the bottom of a file he didn't know existed.

He finds it four levels down: a comparison that assumes something about colour that isn't true for scanned images.

He changes one line.

**It feels like editing that function. It is not.** The original is untouched and everyone else using it is unaffected, forever. Yusuf now holds his own version — of that function, and of the short chain above it. His inspector says: *you diverged three things; the substance you changed is one line.*

He carries on. It works.

### Hitting the wall on purpose

Forty thousand photographs. It takes nine minutes and uses six gigabytes of memory.

Now — and only now — Yusuf has a reason to care about efficiency. Not because a tutorial told him to, or because the language demanded it upfront, but because he ran into an actual wall.

He tightens the part that matters. Explicit ownership, no copying, precise types — the strict end of the same dial, applied to the three functions in the hot loop, and nowhere else.

**And two of the three are already done.** Somebody else needed the same tightening two years ago, did the difficult work, and it lives on the same definitions. He takes it.

> This is the part with no equivalent today. Right now, the effort someone spends optimising a library is trapped inside their program and dies with it. Here it accumulates on the thing itself, available to everyone who ever conjures it afterwards.

Nine minutes becomes forty seconds.

### Running it

He runs it.

Almost nothing compiles, because almost nothing is new — the hashing library, the image decoders, everything underneath: all of that was compiled once, by somebody, and the fingerprints match. What compiles is his own code and the four functions he diverged.

And when it starts, **it can only see his photographs.** Not because he wrote a security policy — he wrote nothing. The system already worked out exactly which pieces went into building this thing, and that same list became the world it runs in. It cannot read his email, cannot reach the network, cannot see the rest of his disk. Those things are not forbidden to it. They are absent.

> He did not configure this. He could not have got it wrong. It fell out of a calculation performed for an entirely different reason.

### Changing it while it runs

Halfway through a forty-thousand-photograph run, he notices the threshold is slightly too aggressive.

He changes it. **The running program picks it up and keeps going.** No restart, no losing thirty minutes of progress.

He can do that because the system checked first: only the inside of a function changed, no stored data changed shape, no boundary moved — therefore safe to swap live. It knew before he tried. Had he changed something structural, it would have said so, and told him what would be required instead.

### Publishing

He is happy with it. He publishes.

There is no packaging step. No version number to invent. No manifest to write, no dependency list to declare — the list would be redundant, because the exact closure is already known and pinned. No repository to create.

**What he publishes is a name pointing at a fingerprint**, plus the handful of definitions that are genuinely his: his own code, and the four small divergences.

His secrets — the credentials for the backup service he files the extras to — are not in it. They never were. The graph holds a handle meaning *"the value behind this,"* and the value itself sits in a private compartment that never left his machine. He can rotate it tomorrow without any of this changing.

### Somebody else

A week later, Rin finds it.

She clicks once. Nearly everything is already on her machine, shared with a dozen other things she runs. What arrives is Yusuf's small divergent chain, and it starts.

It is frozen exactly as he left it. If somebody changes one of those underlying functions next month, that is a different fingerprint and a different thing, and it cannot reach her.

**She wants one difference:** her scanned negatives need a different threshold. She changes it. That is now *her* version, costing one small record, affecting nobody.

**Then she measures.** On her hardware, with her workload, a different setting is meaningfully better. She attaches the measurement to the value — with its conditions, and reproducible by anyone who doubts it.

Yusuf sees it. He was wrong, on that hardware. He takes her number as the new default.

**And the fix he made four levels down**, in that stranger's hashing library, gets offered upstream. The maintainer sees precisely one line — no formatting noise, no naming noise, just the structural change. They take it. Yusuf's version and theirs become the same fingerprint, and are now one thing.

Somewhere, unremarked, three other people who had independently made the same fix converge onto it too.

---

### What that was

Read it back and count what never happened.

No project setup. No build configuration. No package manager. No dependency resolution, and no conflicts to resolve. No install steps. No version numbers. No lock file. No container. No `works on my machine`. No forking a library to change one line. No maintaining that fork. No vendored copies. No security policy to write. No restart to apply a change. No packaging. No release process. No documentation search, because the explanations were attached to the things they explain.

And count what did happen, which cannot happen today.

**Someone got a working thing on the first evening in a language famous for its steep entry.** **Effort spent tightening code accumulated on the thing itself instead of dying inside one program.** **A stranger's improvement flowed back and merged with three other people's identical work.** **A measurement corrected an author's wrong default, with evidence anyone could check.** **A fix four levels down cost one line and no ongoing maintenance whatsoever.**

None of that is exotic technology. Every piece of it exists today, in production, built by people solving unrelated problems.

**It has simply never been assembled.**

---

# Part Five — Why now, and what it means

## 17. Every piece already exists

This is what separates a plan from a daydream. **Not one component here needs inventing.** Each is running in production somewhere, built by people solving a different problem, who accidentally built the foundation.

- **Fingerprint-based identity, in a working programming language.** A language called Unison identifies every definition by the fingerprint of its structure, with names stored separately as labels. Dependency conflicts don't merely become rare there — they *cannot occur*, structurally. It reached version 1.0 in late 2025, with the team running their own production infrastructure on it.

- **Whole-tree-visible, nothing-downloaded.** Google, Microsoft and Meta, independently, at billion-line scale. Parts of Microsoft's work were folded back into git itself.

- **Reversible slicing of a codebase.** A tool called josh already presents arbitrary compositions of code as if they were independent projects, with changes flowing correctly back to the original. Written in Rust.

- **Optional parts that stay unbuilt.** The Linux kernel has done this for thirty years across forty million lines. You declare what you want; only that compiles.

- **Fine-grained change analysis.** The Rust toolchain already publishes a complete structured description of what a component offers, and existing tools already compare two versions to find exactly what changed.

- **Splitting a component to resolve a conflict.** Rust developers already perform this refactor by hand, constantly, as an established convention. Automating it is automating an existing chore, not inventing a technique.

- **A language morphing its own form.** Rust already ships different syntax variants that interoperate within a single program. That is a language changing shape without splitting its ecosystem — a decade in production.

- **The substrate stabilising.** Rust became a permanent, non-experimental part of the Linux kernel in 2026, with some areas moving to Rust-only for new code.

Eight foundations. Eight groups of people who never coordinated.

**Nobody has put them in the same room.** That is the entire remaining work.

### And there's a reason it's now

The design was always plausible and always impractical, because it requires a great deal of careful, patient authoring work — writing the adaptation layers, the tightened versions, the bridges.

That specific kind of labour has recently become dramatically cheaper. Meanwhile the current system demands the one thing humans are worst at and machines are best at: memorising ten thousand arbitrary compatibility facts.

The idea didn't get better. The cost of the missing work collapsed.

---

## 18. It behaves like a living system

The parallels here are not decoration. Repeatedly, a decision reached on engineering grounds turned out to already have a biological name — and the biology then predicted something not yet worked out.

**Fingerprints are the genes; names are the appearance.** The substance is inherited. The label is local, visible, and disposable.

**Changes to a running program don't reach the shared store.** Biology has a precise term: the *Weismann barrier*. Changes to your body do not propagate to your children — learn a language, break an arm, get a tattoo; none of it is inherited. Only changes to the genes are.

The same rule here: **the shape you drew, and the name you gave it, stay yours. Only the substance goes home.**

That is not bureaucracy — it is what stops everyone's environment drifting into an unreproducible one-off. Which is precisely the failure that killed earlier systems built on similar ideas: you could change anything live, but your machine slowly became a snowflake nobody could reconstruct.

**Grafting is how bacteria share traits sideways.** Not by inheritance from a parent, but laterally, across species — which is how antibiotic resistance spreads. Taking a capability from an unrelated piece of software is the same move.

**Renaming is a silent mutation.** In genetics, a change to the code that doesn't change the result. Here: the name changes, the fingerprint doesn't, nothing downstream notices.

**Settings are gene regulation.** The same definition behaves differently depending on the environment it's placed in. Which means "what does this actually do?" is only answerable relative to a context — and that isn't a defect. It is exactly what biology means when it says there is no such thing as a trait outside an environment.

**Disuse is how things die.** Something nobody uses is never built, costs nothing, and quietly stops being part of the living system. Nothing needs deleting.

### Why the pattern is evidence

**A model that keeps making correct predictions in a field it wasn't built for is probably carved along the real joints.**

And there's a second, harder piece of evidence. When three organisations independently invent the same solution without coordinating, biologists call that *convergent evolution* — and treat it as the strongest available sign that a solution is forced by the environment rather than arbitrary. The eye evolved independently dozens of times. The whole-tree-visible architecture evolved independently three times in fifteen years.

The components here keep turning out to already exist, built by people who weren't trying to build this.

**That is what it looks like when an idea is due.**

---

## 19. What actually changes

**Version conflicts that turn out not to exist.** The afternoon you lost was to a claim, not a fact.

**A whole operating system becomes a saved selection.** Forkable, comparable, shareable, and sharing its storage with every other selection. The same kind of object as a single program.

**Contributions that can be accepted almost without review** — because an addition that only activates under conditions you don't meet is structurally incapable of breaking you.

**Everything readable, and cheap.** You pay only for what you actually change. No component is ever a sealed box again.

**Exact reproduction with nothing to declare.** No requirements list to write, get wrong, or lie about. It works on your machine because it *is* the same thing.

**You stop searching the internet to find out about your own computer.** Every setting, its explanation, its history, and what other people measured, all reachable by asking the machine in front of you — current by construction, because there is no separate copy to go stale.

**One kind of person.** No developers and users as separate populations with separate tools. Just people at various distances from any given piece — and the distance is one keystroke. Someone who hits a bug is *already* looking at the thing that caused it.

### And the physical waste, item by item

Everything in section 3 was one problem wearing many costumes: **no way to tell whether two things are the same.** One structural change answers all of it at once, and none of these is a separate feature anyone had to design.

**The five browser engines become one.** Not because anyone coordinated a shared version, and not because five teams agreed on anything — but because five identical things compute to one fingerprint and are therefore, unavoidably, one stored thing. Where two applications genuinely need different versions, they get different versions, side by side, no conflict, no hell. **Sharing without agreement is the thing that was never possible before.**

**Five copies of the dependency folder become one.** Same reason. The same package at the same version, in five projects, is one fingerprint.

**A dozen rebuilds of the same library become one build.** Compiled output is stored by fingerprint too, so the second project to need it does not build it — and neither does the two-thousandth, anywhere on earth.

**Forty container images become forty small differences.** There is no operating system to ship, because there is no sealed box that needs one packed inside it. A service is its closure, and its closure shares everything it has in common with every other service, automatically.

**The registries stop sending the same bytes.** You cannot request something you already have, because "do I already have this?" is a fingerprint comparison rather than a filename and a hope. What crosses the network is the difference, and only the difference.

**Automated builds stop starting over.** Caching stops being keyed to a whole project — where one trivial change throws everything away — and becomes keyed per definition. Change one line and one thing rebuilds. The rest was already valid, and stays valid.

**Update downloads become the actual change.** Not a fresh copy of an application because three lines moved inside it.

**And the security cost inverts.** *Where are the copies?* stops being an investigation and becomes a lookup, because there are no hidden copies — there is one of each thing, and an exact list of what reaches it.

**Then the measurement problem dissolves too.** Remember that nobody can tell you how much of the world's stored software is redundant, because the architecture destroys that information at build time. Here the information is the architecture. You can simply ask.

> **And note what did *not* happen.** Nobody optimised anything. There is no deduplication pass, no compression stage, no cleanup tool, no shared-runtime initiative, no committee agreeing on standard versions. The duplication does not get removed — **it never gets created**, because two identical things were never two things in the first place.

That is what makes this structural rather than an efficiency drive. Efficiency drives are work you must keep doing. This is a property of the arrangement.

---

### And the one that justifies the whole effort

The next great programming language does not demand that the world be rewritten. It demands a new surface over the same substance. We stop rewriting everything every ten years, because we made *change itself* the basic operation, instead of hiding it inside sealed boxes called compilers and packages and programs.

---

## 20. Where to begin, honestly

**Start small and useful.** Read real code, fingerprint it, and prove that identity works. Then extract a single behaviour from something large and show the result shrink while still working. Then cache compiled pieces by fingerprint so only genuine changes rebuild.

Those three steps produce a tool worth using on its own merits, regardless of whether anything above them is ever built. Everything else — the views, the live values, the sandboxing, the sharing — comes after, and each stage stands alone.

### The risk is not the engineering

Every layer has a working precedent somewhere. This is assembly, not invention. The genuinely unbuilt parts are few and tractable.

**The risk is entirely adoption**, and it sits in one place: the first person to use this must come out ahead **on day one**, alone, with no community behind them and nothing rewarding them for participating.

That is the only bar that matters. Systems that cleared it succeeded; systems that asked people to invest first for a payoff later did not.

Build the thing that helps the first user immediately. Everything after that follows.

---

## 21. The shape of the whole thing

Read it back and there is one move, repeated:

**Replace a guess with a calculation.**

| Question | Today | Here |
|---|---|---|
| Which version is this? | Guessed from a name | Computed from contents |
| What does this need? | Guessed from a declared list | Computed by following actual use |
| Does this need rebuilding? | Guessed from timestamps | Computed from fingerprints |
| Do these two conflict? | Guessed from version numbers | Computed from what actually differs |
| What may this program touch? | Guessed by a hand-written policy | Computed from what built it |
| Is this update safe to apply live? | Guessed by trying | Computed in advance |
| Who is affected by this flaw? | Investigated for months | Looked up |
| What does this setting do? | Searched for on the internet | Asked of the machine itself |
| Is that advice still true? | Hoped | Reproduced |
| Are these two things the same? | Unanswerable | The whole basis of the system |
| How much of this is a duplicate? | Not measurable | Zero, by construction |

Every one of those guesses currently costs the industry an enormous amount of human life. Every one became calculable the moment things had stable identities.

That is the whole idea. Everything else is engineering.

---

## 22. The ask

We spent thirty years building extraordinary tools and then hobbled them with an organisational model inherited from tape drives. We identified things by labels that change. We described compatibility at a resolution too coarse to be true. We made isolation the default and connection the chore. And we built a professional culture that mistook climbing the resulting walls for evidence of intelligence.

It cost an ocean of hours, and an unknowable number of people who would have been brilliant at this and were told, in a thousand small ways, that the toll booth was the job.

Everything needed to end it is on the table. The identity model works and is shipping. The storage architecture was invented three times independently. The slicing tool exists and is written in Rust. The optional-parts system has run the Linux kernel for thirty years. The change analysis ships in the Rust toolchain today. And the foundation — Rust in the kernel — stopped being an experiment months ago.

This is not a moonshot. It is an assembly job that got easier every year for a decade while everyone was looking somewhere else, and is now merely *hard* rather than impossible.

The open source community built every one of these pieces. It should be the one to put them together.

**Stop managing software. Start growing it.**

