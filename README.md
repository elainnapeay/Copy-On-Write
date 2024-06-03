# Copy-On-Write
Artifacts for COW group project for CSCI 4727


===== COW =====
- system-level optimization: uses less memory by keeping single copy of each unique page
- point: avoid copying costs, reduce RAM use, reduces amount of time spent in fork() - challenges: how to avoid freeing memory until last process is finished (need more bookkeeping),
mimic page faults
- strategy: small steps, work on subsets of problem
- instead of copying a process' memory, we just want to copy its page table (child has carbon copy of page table, not memory)
- fork() does its copying in vm.c
- instead of allocating memory and copying onto it (commented out), copy page table entries
- change mappages to map "pa" (physical address) pulled out of parent's page table instead of mem *pa
- all of the pages in parent page table are writable, need to write protect
- need to clear write flag in child and parent bc we don't want child to
have access. trying to mimic separate copies *test* both mappings of the page are read-only
- any store into a write-protected page will cause a page fault - now we want to handle these page faults in the usertrap() in trap.c
- we want to allow reading, but catch write faults - cowfault function makes copy of page
- needs current page table, and virtual address that was faulted on *test* timestamp: 35 minutes (check walk)
- maintain reference count in kalloc.c
  
- add decrements to proper files
*test* need to fix two processes running simultaneously
- copyout looks up virtual address and translates to physical address without MMU, so write fault is not caught
- modify copyout to check for write permission
- modify uvmcopy():
- map parent's physical pages into child
- change PTE to read-only
- PTE (page table entry) of COW marked using the RSW (reserved for software) bit (ignored by memory management unit that translates virtual addresses to physical)
- modify usertrap():
- recognize/handle page faults
- add new method to handle COW faults
- kill process when a COW page fault occurs and there's no free memory
- reference count (number of user page tables that refer to that physical page) is recorded for each physical page using array
(shifting 12 bits, >>12, equivalent to dividing by 4096 → size of physical page)
- count set to 1 when kalloc() allocates
- increments when fork() causes child to share page, new function in kalloc.c - decrements when any process drops page from its page table in kfree()
- released when count is 0, kfree() places page back on free list
- modify copyout() to use same scheme as page faults when it encounters COW page
- call added method from usertrap() to make sure you don't write to a COW page
