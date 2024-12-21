# alligator

Attempt to implement malloc-like free list allocator.

The memory to manage may be supplied by the user, or the user-specified amount of memory is borrowed from malloc (for now, until mmap is implemented).

Once we have a buffer of memory to manage on user's behalf, it is "formatted" so as to contain a single free block.

A block consists of a *header* (8 bytes) that tracks the size of the associated chunk and its status (free or occupied). Header is followed by a *chunk* that represents the available block's space. A block is closed by a *footer* (8 bytes) which is a repeated header (contains the chunk's size and status).

So, a block consists of a chunk surrounded by a header on the left and a footer on the right. However, if the block is free, then there is also a *node* embedded at the beginning of the free chunk space.

Free blocks in the buffer are tracked by a doubly-linked list (aka free list), which is a sequence of nodes, with each free chunk temporarily hosting a node at its beginning.

A node takes 16 bytes (it has `next` and `prev` pointers), which dictates the minimal chunk size (16 bytes - so a node fits); this in turn dictates the minimal block size of 32 bytes (size of header, node and footer).



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
