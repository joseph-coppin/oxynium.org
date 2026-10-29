# Examples

These are a few simple examples which showcase some of the language's features.

## Hello, World!

```oxynium
print("Hello, World!")
```

## Fibonacci

```oxynium
const N = 10;

def fib (n: Int) Int {
    if n <= 1 {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}

print(fib(N).Str()); // 55
```

## FizzBuzz

```oxynium
def main() {
    for i in range(1, 101) {
        print(fizzbuzz(i));
    }
}

def fizzbuzz (n: Int) Str {
    if n % 3 == 0 && n % 5 == 0 {
        return "FizzBuzz";
    }
    if n % 3 == 0 -> return "Fizz";
    if n % 5 == 0 -> return "Buzz";
    return n.Str();
}
```

## Generic classes
```oxynium
class LoggedQueue<T> {
    backing_list: List<T>,
    
    def make_empty<S>()
        -> new LoggedQueue<S> { backing_list: List.empty!<S>() },
    
    def push(self, item: T) LoggedQueue<T> {
        self.backing_list.push(item);
        print("added item " + self.len().Str());
        return self;
    }
    
    def pop(self) T {
        if self.len() < 1 -> panic("pop on empty list");
        let val = self.backing_list.remove_at(0).unwrap();
        print("popped item " + self.len().Str());
        return val;
    }
    
    def len(self) ->
        self.backing_list.len()
}

def main() {
    let q = LoggedQueue.make_empty!<Int>()
                       .push(5)
                       .push(6);
    
    print("process: " + q.pop().Str()); // 5
    print("then: " + q.pop().Str()); // 6
}

```