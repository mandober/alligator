# alligator

Attempt at implementing malloc-like free list allocator.

The memory to manage may be supplied by the user, or the user-specified amount of memory is borrowed from malloc (for now, until mmap).

Once we have a buffer of memory to manage on the user's behalf, the buffer is "formatted" so as to contain a single (free) block.

A *block* consists of a *header* (8 bytes) that tracks the size of the associated chunk and its status (free or occupied). Header is followed by a *chunk* that is the available space in a block. A block is closed by a *footer* (8 bytes), which is just the repeated header (also contains the chunk's size and status).

So, a block consists of a chunk surrounded by a header on the left and a footer on the right. However, if the block is free, then there is also a *node* temporarily embedded at the beginning of the free chunk's space.

Free blocks in the buffer are tracked by a doubly-linked list (aka free list), which is a sequence of nodes, each of which located at the beginning of the chunk (right after the header).

A node takes 16 bytes (it has `next` and `prev` pointers), which dictates the minimal chunk size (16 bytes - so a node fits), which, in turn, dictates the minimal block size of 32 bytes (size of header, node and footer).



```
header
│ node                                       footer
↓  ↓                                           ↓
┌──┬──┬───────────────── B₀ ───────────────────┬──┐
│H₀│N₀│∙∙∙∙∙∙∙∙∙∙∙∙∙chunk size∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙│F₀│
└──┴──┴────────────────────────────────────────┴──┘

┌──┬──┬───── B₁ ─────────┬──┰──┬────── B₂ ─────┬──┐
│H₀│N₀│∙∙∙∙∙∙∙rem∙∙∙∙∙∙∙∙│F₁┃H₁│∙∙∙∙∙∙req∙∙∙∙∙∙│F₀│
└──┴──┴──────────────────┴──┸──┴───────────────┴──┘
↑  ↑                     ↑  ↑  ↑               ↑
│ node                   │  │ ptr            footer
header           footerNew  headerNew
```
