# Sample

Here is a bit of code manually converted to C3 from C.

```

struct Node
{
    uint hole;
    uint size;
    Node* next;
    Node* prev;
}

struct Footer
{ 
    Node *header;
}

struct Bin  
{
    Node* head;
}

struct Heap  
{
    size start;
    size end;
    Bin* bins[BIN_COUNT];
}

const uint OFFSET = 8;

/**
 * @require heap != nil, start > 0
 */
void Heap.init(Heap* heap, usize start) 
{
    Node* init_region = cast(start as Node*);
    init_region.hole = 1;
    init_region.size = HEAP_INIT_SIZE - @sizeof(Node) - @sizeof(Footer);

    init_region.createFoot();

    heap.bins[get_bin_index(init_region.size)].add(init_region);

    heap.start = cast(start as void*);
    heap.end   = cast(start + HEAP_INIT_SIZE as void*);
}

void* Heap.alloc(Heap* heap, usize size) 
{
    uint index = get_bin_index(size);
    Bin* temp = cast(heap.bins[index] as Bin*);
    Node* found = temp.getBestFit(size);

    while (!found) 
    {
        temp = heap.bins[++index];
        found = temp.getBestFit(size);
    }

    if ((found.size - size) > (overhead + MIN_ALLOC_SZ)) 
    {
        Node* split = cast(cast(found as char*) + sizeof(Node) + sizeof(Footer) as Node*) + size);
        split.size = found.size - size - sizeof(Node) - sizeof(Footer);
        split.hole = 1;
   
        split.createFoot();

        uint new_idx = get_bin_index(split.size);

        heap.bins[new_idx].addNode(split); 

        found.size = size; 
        found.createFoot(found); 
    }

    found.hole = 0; 
    heap.bins[index].removeNode(found);
    
    Node* wild = heap.getWilderness(heap);
    if (wild.size < MIN_WILDERNESS) 
    {
        uint success = heap.expand(0x1000);
        if (success == 0) 
        {
            return nil;
        }
    }
    else if (wild.size > MAX_WILDERNESS) 
    {
        heap.contract(0x1000);
    }

    found.prev = nil;
    found.next = nil;
    return &found.next; 
}

/**
 * @require p != nil
 */
func void Heap.free(Heap* heap, void *p) 
{
    Bin* list;
    Footer& new_foot, old_foot;

    Node* head = cast(cast(p as char*) - OFFSET as Node);
    if (head == cast(cast(heap.start as usize) as Node*) 
    {
        head.hole = 1; 
        heap.bins[get_bin_index(head.size)].addNode(head);
        return;
    }

    Node* next = cast(cast(head.getFoot() as char*) + sizeof(Footer) as Node*);
    Footer* f = cast(cast(head as char*) - sizeof(Footer) as Footer*);
    Node* prev = f.header;
    
    if (prev.hole) 
    {
        list = heap.bins[get_bin_index(prev.size)];
        list.removeNode(prev);

        prev.size += overhead + head.size;
        new_foot = head.getFoot(head);
        new_foot.header = prev;

        head = prev;
    }

    if (next.hole) {
        list = heap.bins[get_bin_index(next.size)];
        list.removeNode(next);

        head.size += overhead + next.size;

        old_foot = next.getFoot();
        old_foot.header = 0;
        next.size = 0;
        next.hole = 0;
        
        new_foot = head.getFoot(head);
        new_foot.header = head;
    }

    head.hole = 1;
    heap.bins[get_bin_index(head.size)].addNode(head);
}

func uint Heap.expand(Heap* heap, usize sz) 
{
    return 0;
}

func void Heap.contract(Heap* heap, usize sz) 
{
    return;
}

func uint get_bin_index(usize sz) 
{
    uint index = 0;
    sz = sz < 4 ? 4 : sz;

    while (sz >>= 1) index++; 
    index -= 2; 
    
    if (index > BIN_MAX_IDX) index = BIN_MAX_IDX; 
    return index;
}

func void Node.createFoot(Node* head) 
{
    Footer* foot = head.getFoot();
    foot.header = head;
}

func Footer* Node.getFoot(Node* node) 
{
    return cast(cast(node as char*) + sizeof(Node) + node.size as Footer*);
}

func Node* getWilderness(Heap* heap) 
{
    Footer* wild_foot = cast(cast(heap.end as char*) - sizeof(Footer) as Footer*);
    return wild_foot.header;
}
```