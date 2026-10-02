# Reflection

**1.** At around line 115 in src/scanner.rs, I decided a `.` begins a fractional part with the code:

`if self.peek() == '.' && self.peek_next().is_ascii_digit()`

It would only consume the dot when the character after it is also a digit. So if any is not a digit (when `5.` is input for example), the digit would consume `5` first, then when it got to `.` the check I had written on `src/scanner.rs:115` would fail because `peek_next()` would arrive at `.` and consider it a non digit (end of input) and is left unconsumed. `number()` returns after `NUMBER 5` is emitted. `scan_token` then reads `.` on the next call and fall through the `identifier()` function which produces the appropriate scan error: `Character is not part of any token.`.

This falls in line with what was said in Section 1.4 that "`.` begins no token in Kobo" so `5.` produces an expected `NUMBER 5` as it is a valid number token which calls the `number()` function for the corresponding token then when it then runs into the `.`, it is not considered a digit and, in return, producted a scan error.

**2.** I changed the line counter in about two places, these are: `src/scanner.rs:83` for every ordinary newline: `'\n' => self.line += 1` and `src/scanner.rs:98` inside `string()` function for a newline found while scanning a multi-line string: `self.line += 1;`.

For a file ending in two blank lines, each trailing newline is consumed by `src/scanner.rs:83` (seen above) and increments `self.line` by 1, but no token follows either newline. `EOF`'s line comes from `src/scanner.rs:35`, so it takes the line of the last real token (as written in Section 6.1) and not what `self.line` has picked up (as they are two different things).

This is done so `EOF` will always be reported at the end of the last real token and is not silently changed where every test expects `EOF` to be reported that carries zero information (which may cause an error in the tests, both in tests directory and hidden). `EOF` will only be detected when a real token is detected.

**3.** The only real test that I failed that would have shown in the previous code versions (like commit `2972a269`) before commit `3a81cdab` (when it was fixed) was the previously empty comment detection at `src/scanner.rs:71` so it would produce a fail on "tests/phase-1/valid/comments.kobo" everytime phase-1 tests were ran.

The reason was I could not figure out how to have it detect the comment (double slashes) itself, to move to the end of the line if the comment was detected, and to move on if it was not double slashes (in the case it was a newline, tab, and the carriage return character).

It later took some hours looking it up and figured that I needed to make 2 checks: for checking if it is a `/`, and if it was not a newline so it could just run to the end of the line if this was the case. This allowed me to finish the code block along with the rest of the code.
