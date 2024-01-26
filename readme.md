# perfdata

`perfdata` is a set of tools to automate the recording and collection steps of [sampling profile-guided optimization](https://clang.llvm.org/docs/UsersManual.html#using-sampling-profilers) (SPGO).

## Production ready?

```
This software is highly experimental.
It might not work how you expect.
It might change at any time.
It might never update again.
Proceed with caution.
Use for inspiration.
Not for production.
``````

## Overhead

One great feature of sampling PGO is a reduction in overhead compared to instrumented PGO.
Rather than invoking extra injected code at runtime, the kernel uses CPU features to take samples.
And indeed it is true that `perf record` with properly tuned `--mmap-flush=` adds overhead of only about 2% CPU time.

But `perf record --branch-filter 'any,u'` makes massive dumps of data - about 10 GiB per hour or even more with `--freq=max`.
Compression can save space, but of course this adds more CPU overhead.
Even better, we can use an event filter like `--event=br_inst_retired.all_branches` to reduce the amount of data being gathered and processed.

Once the raw data is recorded it has to be summarized in a format a compiler can understand.

- `perf script` replays a simulation of the recording
- `llvm-profgen` tallies up branch counts in the script
- `llvm-profdata` combines multiple profiles

I made [some patches](./patches/) to `perf`, `llvm-profdata`, and `llvm-profgen` which allow them to pipe data around, avoiding intermediate temporary files completely.
Together with `perf-record` event filtering, this brings CPU overhead down below 5% and the IO overhead down to just a few MB per hour, even when recording with `--freq=max`.
