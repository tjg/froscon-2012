# Ideas for the clever mercenary who’ll never learn Lisp

*This talk is dedicated to those who bring guns to knife fights.*

*And tactical explosives. And smoke bombs. And third parties who think your enemies are their enemies.*


## Note

This talk won't advocate a particular tool. In fact, it's worth
maintaining a healthy dose of skepticism when confronted with
advocacy. Particularly when the advocate isn't institutionally
accountable to you.


## Program in pictures: data-directed programming. 

Once I had to deal with a confusing subproject. It seemed simple, but
our first implementation was wrong. We soon realized that no one could
easily articulate how the thing was supposed to act.

So I sat down with the Product Owner, and we used the language of
basic set theory to describe it. But the spec still turned out buggy.

The next time, I started writing down a simple visual matrix and asked
him to fill it in. He improved it, and so it looked something like:


And under different conditions:


At first, I coded it in the usual, rather ugly way. Then I slapped my
forehead. Why not translate the picture from paper to the screen?

One Lisp tradition is for your code to follow the visual
metaphor. Since we worked in node.js, I transformed it into the
following spec:

```javascript
normalSpec = {
  Ra: {Aa: 'video', Av: 'video', Al: 'video', An: 'video'},
  Rv: {Aa: ' ',     Av: 'video', Al: ' ',     An: 'video'},
  Rl: {Aa: ' ',     Av: ' ',     Al: 'video', An: 'video'},
  Rn: {Aa: ' ',     Av: ' ',     Al: ' ',     An: 'video'},
};

textEnabledSpec = {
  Ra: { Aa: 'text',  Av: 'text',  Al: 'text',  An: 'video'},
  Rv: { Aa: ' ',     Av: 'video', Al: ' ',     An: 'video'},
  Rl: { Aa: ' ',     Av: ' ',     Al: 'video', An: 'video'},
  Rn: { Aa: ' ',     Av: ' ',     Al: ' ',     An: 'video'},
};

// Elided about 30 significant lines of lower-level "plumbing" code,
// which executes both datastructures.
```

This is simple to extend. And if we no longer need one of them, it's
trivial to delete. Spec bugs are simpler to fix.

You can also use it to automatically generate test-cases. And it
could've easily come from the network, as JSON.

This technique wasn't new to me. Once upon a time, I wrote rich-client
GUIs using LispWorks, which has a wonderful GUI language like this.


### A political note

Unfortunately, such simple techniques may result in *"fear,
uncertainty and doubt"*. They appear unusual to others; and easily
enter the realm of politics. You may experience resistance.

To understand this, note that institutions evolve ways to defend
themselves from dangerous technological choices; otherwise they'll
more likely fail to accomplish their roles. Also note that programmers
are trained by society to understand only a very restricted part of
the programming cosmos. (To more reliably produce a software
workforce.)

Combine these two factors, and you can predict political resistance to
anything which deviates from that narrow range.


### Monitoring graphs

For another project, we used Munin, which monitors your systems and
displays colorful graphs. I wrote a little data-driven language which
took counters out of Redis and graphed them:

```javascript
var graph = {type: 'multigraph',
             name: 'GETs',
             roots: [{type        : 'combined',
                      title       : 'GET failures (click for more detailed graphs!)',
                      description : 'Failures GETing from cloud',
                      units       : 'fails/\${graph_period}',
                      subgraphs   : [{type     : 'plain',
                                      category : 'articles',
                                      item     : 'failure'},

                                     {type     : 'plain',
                                      category : 'comments',
                                      item     : 'failure'}]}],

             backings: [{type     : 'plain',
                         category : 'articles',
                         item     : 'success'},

                        {type     : 'plain',
                         category : 'articles',
                         item     : 'failure'},

                        {type     : 'plain',
                         category : 'comments',
                         item     : 'success'},

                        {type     : 'plain',
                         category : 'comments',
                         item     : 'failure'}]}
```

### A machine-unreadable spec

Recently, I had to implement the OpenRTB spec, which is just this PDF
which isn't human-readable. I was certainly not going to tediously
hand-code this. So I just ran the PDF through some sort of pdf2txt,
and then had Emacs record my keystrokes while I massaged part of it
into Clojure datastructures (something like the JavaScript/JSON
examples above). So I could replay that massage over the entire spec.

Now it was completely in a machine-readable and -manipulable
format. About 1000 lines of neatly indented data. So I wrote some
Clojure to generate a perfectly-commented JavaScript skeleton from
it. (The comments came from the human-readable descriptions.)

At that point, all I had to do was customize that JavaScript skeleton,
putting in the proper values.

(I could also use it to help generate some test cases and
documentation browser.)


### Concluding thoughts

Were these examples of data, or code? In the Lisp perspective, the
line between the two is blurred — if there is a real line. Data is
powerful. You can interpret in different ways. (As code, tests, etc.)
If you express a language in it, as in our examples above, you can
operate on code using your powerful data-handling muscles. You can
store it somewhere, or pull it from the network. Many possibilities
for the enterprising mercenary.


## Sculpt your program while it runs: incremental development. 

I once had a job where we had a legacy Python codebase. ("Legacy"
means "painful" and "not written by me".) There were no unit tests; to
test the program, you ran it. And waited.

The codebase's first programmer once told me that he built job
security by intentionally writing spaghetti.

This wrecks your design->edit->debug cycles. When you just want to run
a tiny bit of code, it's unnecessary to have to pull in the whole rest
of the program. Much nicer to write layers of small code: with things
broken down into abstractions, which you compose like virtual Lego.

In any case, the modules I wrote were standalone, and designed to run
alongside an interactive REPL. A single keystroke evaluated the
sourcecode I worked on, so I could grow my systems incrementally.

(Note that SQL and Unix have REPLs too — they're just called
interactive prompts. They allow you to interact with a live, growing
system. Rather than stopping and restarting it each time.)

For example, with a single function call, I pushed data safely into
MongoDB. Or queried it at runtime. With all the power of Python.

I believe this led to notably simpler, more composable code.


## Be deadly with one scripting language, and one editor/IDE. 

Commonly, you need to deal with all sorts of "meta" stuff, which isn't
directly about coding. Such as getting data from Riak and stuffing it
into MySQL, munging text, generating bits of code (like print
statements), etc.

Many people do tedious things like:

1. Run curl to get data out of Riak.
2. Pipe it into a file.
3. Open it with Chrome, for readable JSON.
4. Copy bits of it into their clipboard.
5. Paste it into MySQLAdmin.

But why not rely on the power of a general-purpose programming
language? One designed to handle data powerfully and interactively?

(Maybe the first time it takes longer, but next time you've got a
reusable snippet of code which you can take anywhere.)

Python and Ruby are pretty good. And scale faster than cutting 'n
pasting. During an emergency, I needed to transform gigabytes of data
from a MySQL database. So I just loaded it into an Amazon cloud image,
cloned it 20 times, and reimplemented a poor man's MapReduce in Python
over it all.

I would need more than 20 human volunteers to do this.


## Build a strong domain language (without heavy DSLs or macros). 

"Macros" are commonly advertised as Lisp's big power. (They allow you
to safely transform code which Lisp doesn't understand, into a form
which Lisp does. To keep your program fast, this is generally done
before runtime.)

But you don't always need this power. Macros can be less dynamic and
flexible than the data+functions of our previous examples.

One way to look at those examples is to view them as building
languages to express our thoughts in.


## Work on simplicity. 

There's an interesting movement coming from the Clojure world.

"Simplicity Matters" keynote:
http://www.confreaks.com/videos/860-railsconf2012-keynote-simplicity-matters

Rich Hickey (Clojure's creator) advocates simplicity. For your
convenience, he coined the term "complected". At work, you can say,
"This code is complected", meaning that there are multiple concepts
intertwined.

Why? You've probably had the experience of enormous productivity when
you start a project. Then some mysterious force slows your team down,
eventually grinding productivity to a near-halt, regardless how
"Agile" your process is.

Your project may have fallen victim to a force of software
architecture, where your program is unable to grow. Like buildings
which fell under their own weight before innovations like arches.

Illustrated by the rather extreme legacy Python spaghetti code we
discussed earlier, where to understand one part, you're forced to
understand many other parts.

Clojure can be evaluated as a case-study of how pursuing simplicity is
integral to good, truly agile design.


## Program like a gamer.
The Smalltalk community has a feature where you can save an "image" of
your running program, and run it later. Like a virtual machine, or a
game you can save and reload. Some Lisp implementations support this
too.

I never used this technique personally, but I have known people in
finance who shipped their personal trading platforms as LispWorks
images. When it was time for a software update, these apps would just
connect to the server and live-patch themselves.

My understanding is that many Smalltalk programs live as images, not
textual sourcecode.

Since I haven't used this personally, I can't knowledgeably evaluate
this practice. Except for virtual machines like VMWare, which I use
all the time.


## Empower your users with some of your power.

One technique is to give users simple defaults; and a way for "power
users" to decide tradeoffs for themselves.

Macros are one example of allowing users to (safely) mold the
language, to better fit their needs. Such a feature assumes that
you're more of an expert in your domain than some language designer
who may know nothing about your field.

Another example is the (perhaps intimidating-sounding) metaobject
protocol. What does it mean? Meta- means "about", so it's about
objects. A protocol is rules it follows.

So, the metaobject protocol allows you to change the rules of your
object system, giving you guarantees that it will operate in a certain
way (which you can depend on). Because along the way, the OOP system
designers made some tradeoffs for the common case, but which may be
the totally wrong tradeoff for your project.

Perhaps the most common example is having objects backed by a
database. I did this in Python once, with MongoDB.

A more exotic example came from Boeing, which used the metaobject
protocol to design planes. You could quickly change sizes and weights;
and only when you accessed a member, did the system lazily run the
calculations relevant to it.



## Let your tools operate on themselves. 

Example taken from:
http://www.gigamonkeys.com/book/conclusion-whats-next.html

```lisp
CL-USER> (defun add (x y) (+ x y))

CL-USER> (disassemble 'add)
;; disassembly of #<Function ADD>
;; formals: X Y

;; code start: #x737496f4:
   0: 55         pushl	ebp
   1: 8b ec    movl	ebp,esp
   3: 56         pushl	esi
   4: 83 ec 24 subl	esp,$36
   7: 83 f9 02 cmpl	ecx,$2
  10: 74 02    jz	14
  12: cd 61    int	$97   ; SYS::TRAP-ARGERR
  14: 80 7f cb 00 cmpb	[edi-53],$0        ; SYS::C_INTERRUPT-PENDING
  18: 74 02    jz	22
  20: cd 64    int	$100  ; SYS::TRAP-SIGNAL-HIT
  22: 8b d8    movl	ebx,eax
  24: 0b da    orl	ebx,edx
  26: f6 c3 03 testb	bl,$3
  29: 75 0e    jnz	45
  31: 8b d8    movl	ebx,eax
  33: 03 da    addl	ebx,edx
  35: 70 08    jo	45
  37: 8b c3    movl	eax,ebx
  39: f8         clc
  40: c9         leave
  41: 8b 75 fc movl	esi,[ebp-4]
  44: c3         ret
  45: 8b 5f 8f movl	ebx,[edi-113]    ; EXCL::+_2OP
  48: ff 57 27 call	*[edi+39]   ; SYS::TRAMP-TWO
  51: eb f3    jmp	40
  53: 90         nop
; No value
```

Now add optimizations to the "add" function, and see what it compiles
to:

```lisp
(defun add (x y)
  (declare (optimize (speed 3) (safety 0)))
  (declare (fixnum x y))
  (the fixnum (+ x y)))

CL-USER> (disassemble 'add)
;; disassembly of #<Function ADD>
;; formals: X Y

;; code start: #x7374dc34:
   0: 03 c2    addl	eax,edx
   2: f8         clc
   3: 8b 75 fc movl	esi,[ebp-4]
   6: c3         ret
   7: 90         nop
; No value
```

## Piggyback on strong systems.

I've heard of Clojure dialects targeting:
- Java
- .NET
- JavaScript
- Python

Clojure programmers expect to assimilate many advantages (and
disadvantages) of their host platforms. For example, Java is an
enormous ecosystem; but Python seems more attractive for those
commandline scripts where immediate startup time is valuable.

It is worth considering how you can turn the systems you encounter to
your advantage, even if they at first appear hostile to you.
