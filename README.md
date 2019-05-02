# ForceCheck

A Nimble package to stop functions from allowing Exceptions to bubble up.

# Usage:

`forceCheck` takes the place of the existing raises pragma. The macro copies your function, adds a blank raises pragma, replaces every raises in the copy with a discard, and then has that compiled with your actual function. If Nim has an error, your function had a potential bubble up. Your actual function does have a raises pragma added as well, and the copy is removed from the binary as dead code by Nim.

`forceCheck` can be used with asynchronous functions, as long as the pragmas are in the following order: `{.forceCheck: [], async.}`. Every async function can raise `Exception`, making a raises worthless. That said, when `forceCheck` copies your function, it removes `async` from the copy and replaces every `await` with `waitFor`, allowing proper bubble-up detection. Since there's no value in adding raises to your actual function, it isn't done. To correct for the lack of raises, a second function copy is made, which is synchronous but does still have all its raises, and that copy has the raises pragma.

As some exceptions shouldn't be handled, irrecoverable exceptions, it's possible to specify those:
```
{.forceCheck: [
    recoverable: [
        KeyError,
        ValueError
    ],
    irrecoverable: [
        OSError
    ]
].}
```
The irrecoverable exception must be specified in every function it bubbles up through, as well as the function that raises it.

`forceCheck` also forces user to handle bounds checks. If these are disabled by the compiler, the code will still be generated, but the actual execution will lead to undefined behavior. If you use the C++ backend, this extra code will be zero cost except for a few extra bytes of disk space. In the case that `forceCheck` thinks it found an uncaught bounds check, but didn't, `fcBoundsOverride` is available, which will disable forced checking of all found 'violations'.  It can be used as a block statement or a pragma.

This library also includes `fcRaise`, a custom version of `raise` to get around https://github.com/nim-lang/Nim/issues/11118 without affecting the raises pragma. It was originally going to be an independent macro, yet was merged for compatibility with the `forceCheck` macro. It's a drop in replacement for `raise`s in `raise e` situations, where `e` is an Exception capture via `as`.
