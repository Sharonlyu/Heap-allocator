# Heap-allocator


implicit
--------
My implicit allocates memory by searching the heap for free blocks via an implicit list.
Every block is at least 8 bytes, which is the header size. The header stores the payload size
except for the last 3 bits. I use the last one bit to show the free status. 1 is for in-use and
0 is for free. I have 2 global variables: segment_start and segment_end, which specify the range
of given heap.

I have 4 helper functions: roundup from bump.c, get_payload_size (applying a bit mask to get the value
stored in header other than the last bit), is_alloc (applying a bit mask to check the last bit),
split_block(splitting a chuny block by changing their header information)
.

For mymalloc,I choose to iterate through all the blocks until my current address of the payload
reaching the segment_end. If the given size is 8 bytes greater than the round-up requested size,
then it will split the block into an used one with requested size and a free block with the rest of the
size after taking account of the header size. I mark the block as allocated by turning on the last bit
of the header.

For myfree, I apply a bit mask to turn off the last bit of choosen block.

For realloc, if the the size of the old payload is enough, then I choose not to move the contents and even split
the block if necessary. Othewise, I call mymalloc to allocate a block with the new size and move the old contents
to that region.

explicit
--------
My implicit allocates memory by searching the heap for free blocks via a doubly-linked-list, which
uses the first 16 bytes of each free payload for prev/next pointers. The header works the same as
the implicit allocator. I uses a struct, called node, which has 2 void* prev and next.I have 3 global
variables: segment_start and segment_end, which specify the range of given heap. free_list is a void*
which is the pointer to the first node.

I have 3 helper functions: roundup from bump.c, get_payload_size (applying bit mask to get the value
stored in header other than the last bit), is_alloc (applying bit mask to check the last bit),
split_block(splitting a chuny block by changing their header information),
get_header_ptr (applying pointer arithmetic to get the address of the header given the payload address),
add_free_node and remove_free_node (rewiring the list to add/remove free block from the free list),
coalesce(changing the header information of the current block and remove its free neighbor from the list)

For mymalloc,I choose to iterate through only free blocks until my current address of the payload
is NULL, meaning it reaches the end of the doubly-linked-list. If the given size is 32 bytes greater than
the round-up requested size,then it will split the block into an used one with requested size and a free
block with the rest of the size after taking account of the header size. I mark the block as allocated by
turning on the last bit of the header. Also, I remove this block from the list.

For myfree, I call the helper function coalesce to merge the right free block with the current block. Then,
I apply a bit mask to turn off the last bit of choosen block and add this block to the list.

For realloc, I coalesce the free neighbor of the current block first. if the updated payload is enough, then I
choose not to move the contents and even split the block if necessary. Othewise, I call mymalloc to allocate a
block with the new size and move the old contents to that region.
