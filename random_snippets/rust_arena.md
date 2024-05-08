Talked with someone about allocators. Implemented an arena and thought it's cute
how borrow checker can verify we are using it correctly. Enjoy:

```rust
pub struct Allocator<'a>(&'a mut [u8]);

impl<'a> Allocator<'a> {
    pub fn new(buffer: &'a mut [u8]) -> Self {
        Self(buffer)
    }
    
    pub fn alloc(&mut self, n: usize) -> &'a mut [u8] {
        let (res, rem) = std::mem::take(&mut self.0).split_at_mut(n);
        self.0 = rem;
        res
    }

    pub fn try_alloc(&mut self, n: usize) -> Result<&'a mut [u8], ()> {
        match n < self.0.len() {
            true => Ok(self.alloc(n)),
            false => Err(()),
        }
    }

    // Don't know the relevant nomenclature, so the name is likely weird. This
    // allows us to reuse parts of the buffer without freeing *everything*.
    pub fn frame(&mut self) -> Allocator<'_> {
        Allocator(self.0)
    }
}
```

Example showing it in action:

```rust
let mut buffer = vec![0; 10];
let mut allocator = Allocator::new(buffer.as_mut());
let b1 = allocator.alloc(2);
let b2 = allocator.alloc(2);
b1[0] = 1;
b2[0] = 2;

let b3 = allocator.frame().alloc(1);
b3[0] = 3;
std::mem::drop(b3);
let b4 = allocator.frame().alloc(2);
assert_eq!(b4[0], 3);
b1[1] = 11;
b4[1] = 4;

println!("{:?}", buffer.as_ref());
```
