# ref-cache-vec
A reusable `Vec<&T>`

## Motivating usecase

Simplifying from a real world usecase,
sometimes you have data that has a limited lifetime
here this is simulated with scopes,
perhaps the "real" data is buffers that are being reused-
such as the cache itself-,
or borrowed from a `Guard` on a `Mutex`.
A motivating purpose of this would be processing large amounts of data in a loop indefinitely,
with attempts to avoid re-allocating large buffers.
```rs
#[derive(Default)]
struct Cache<'a> {
    data: Vec<&'a str>,
}

fn with_cache<'a, T: AsRef<str>>(cache: &mut Cache<'a>, data: &'a [T]) {
    // note: we want to completely clear the cache every time-
    // we don't care about the prior contents, just the backing storage.
    cache.data.clear();
    for datum in data {
        cache.data.push(datum.as_ref());
    }
}

fn base() {
    let mut cache = Cache::default();

    {
        let cached_data = ["a".to_owned(), "b".to_owned(), "c".to_owned()];
        // error[E0597]: `cached_data` does not live long enough
        with_cache(&mut cache, &cached_data);
        //                     ^^^^^^^^^^^^ borrowed value does not live long enough
    }
    
    {
        with_cache(&mut cache, &["d"]);
        //         ---------- borrow later used here
    }
}
```
