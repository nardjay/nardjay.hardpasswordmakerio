Vault

Generate strong passwords and passphrases in your browser.

A tiny, self-contained password generator with real entropy math, honest crack-time estimates, and a genuine "nothing leaves your device" guarantee. No install. No build step. No backend.


TABLE OF CONTENTS

- What is Vault?
- Why
- Features
- Quick start
- The three modes
  - Single
  - Bulk
  - Check
- Understanding the numbers
  - Entropy
  - Crack time
  - Strength labels
- The options
  - Length and words
  - Character sets
  - Presets
- How it works
  - The randomness
  - The entropy math
  - The check analysis
- Keyboard shortcuts
- Project structure
- Running locally
- Deploying to GitHub Pages
- Browser support
- Security notes
- FAQ
- Contributing
- Credits
- License


WHAT IS VAULT?

Vault is a password generator that runs entirely in your browser. Open it, get a password, copy it, use it. No account, no sign-up, no network requests.

It generates three kinds of secrets:

- Random passwords (mixed letters, digits, symbols)
- Passphrases (memorable words)
- Numeric PINs (if you need them)

It also checks any password you paste in, tells you how strong it really is, and gives you an honest estimate of how long it would take to crack.


WHY

Most password generators do one of three things wrong:

1. Use Math.random() instead of a cryptographic source
2. Lie about strength (call a 6-character password "strong")
3. Require an account, an upload, or a network round-trip

Vault doesn't do any of that.

- Randomness comes from crypto.getRandomValues — the same source the browser uses for TLS.
- Strength numbers are real math, not vibes.
- Nothing is transmitted. You could pull the network cable after the page loads and it would still work.


FEATURES

- Three modes: Single, Bulk, and Check
- Real cryptographic randomness (crypto.getRandomValues, with rejection sampling)
- Live entropy calculation in bits
- Honest crack-time estimates (assumes 10^12 guesses/sec, modern GPU rig)
- Password check analysis: length, variety, repeats, sequences, dictionary words, keyboard patterns, years, common passwords, leet substitutions
- Bulk mode: up to 500 passwords at once, download as .txt
- Presets for common needs: Basic, Strong, Maximum, Passphrase
- Passphrase mode using a curated 400-word list
- Exclude look-alikes option (no 0/O, 1/l/I, etc.)
- Copy buttons everywhere, with fallback for older browsers
- Settings persist to localStorage
- Fully offline once loaded
- No dependencies, no frameworks, no CDN


QUICK START

Use the hosted version

Open the live site. The first password is generated automatically. Copy it. Done.

Or run it yourself

1. Download index.html.
2. Open it by double-clicking — no server needed.
3. Start generating.

That's it.


THE THREE MODES

Single

One password at a time. The display updates live as you change settings. The stats panel shows:

- Entropy in bits
- Search space (total possible combinations)
- Crack time at 10^12 guesses/sec
- A strength meter and label

Click the copy icon or press C to copy. Click regenerate or press R to make a new one.

Bulk

Generates a list of passwords using the same settings. Set the count (1 to 500), click Generate, then:

- Click any password to copy it
- Click the small copy icon next to any row
- Click Download .txt to save them all as a text file

Useful when setting up multiple accounts at once, or when provisioning devices.

Check

Paste any password to analyze it. Vault runs a full analysis and shows:

- A 0–100 score
- The strength label and color
- The entropy in bits
- Crack time at 10^12 guesses/sec
- A list of specific issues found
- A list of things that are good about it

Everything runs locally. Nothing is sent anywhere. The input is masked by default; click Show to reveal.


UNDERSTANDING THE NUMBERS

Entropy

Entropy is measured in bits. It represents how uncertain an attacker is about your password.

- 40 bits — weak, crackable by a determined hobbyist
- 60 bits — OK for low-value accounts
- 80 bits — strong, safe for basically anything personal
- 120+ bits — overkill, but harmless

Random passwords:
    entropy = length * log2(charset size)

Examples:
    16 chars, all four sets on: 16 * log2(94)  = ~105 bits
    20 chars, all four sets on: 20 * log2(94)  = ~131 bits
    12 chars, lowercase + digits: 12 * log2(36) = ~62 bits
    8  chars, digits only:        8  * log2(10) = ~27 bits

Passphrases:
    entropy = words * log2(wordlist size)

Vault's wordlist has 400 words, so each word adds about 8.6 bits:
    4 words: 34 bits  (too weak)
    5 words: 43 bits  (OK)
    6 words: 52 bits  (good)
    8 words: 69 bits  (strong)

PINs:
    entropy = length * log2(10)
    4 digits: 13 bits
    6 digits: 20 bits
    8 digits: 27 bits

Crack time

Vault assumes an attacker with a modern GPU rig capable of 10^12 guesses per second. This is realistic for offline attacks against poorly hashed passwords.

Formula:
    seconds = 2^bits / 2 / 1,000,000,000,000

The division by 2 accounts for the average case — attackers expect to find the password after searching half the space, not all of it.

This is not the worst case. It's not the best case. It's the honest middle.

Strength labels

    0–27 bits   Very weak   red
    28–39 bits  Weak        orange
    40–59 bits  Fair        yellow
    60–79 bits  Strong      light green
    80–119 bits Very strong green
    120+ bits   Overkill    violet


THE OPTIONS

Length and words

For random passwords and PINs, use the Length slider (4 to 64). Longer is always better than more character variety.

For passphrases, use the Words slider (3 to 10). Each word adds about 8.6 bits.

Character sets

Four toggles, all on by default:
- Lowercase (a–z)
- Uppercase (A–Z)
- Digits (0–9)
- Symbols (!@#$%^&*()-_=+[]{};:,.<>?/~)

At least one must be on. Turning sets off shrinks the search space.

Exclude look-alikes

Removes characters that are easy to confuse when reading or typing:
    0 O o I l 1 | ` ' " { } [ ] ( ) / \

Use this when you'll be reading the password off a screen and typing it somewhere else.

Presets

Four one-click presets:

- Basic — 16 chars, no symbols. For places that reject symbols.
- Strong — 20 chars, all sets on. The default suggestion.
- Maximum — 32 chars, all sets on. Overkill, but harmless.
- Passphrase — 6 words. For people who need to remember it.


HOW IT WORKS

The randomness

Vault never uses Math.random(). Every random number comes from:

    crypto.getRandomValues(new Uint32Array(1))

That's the same CSPRNG the browser uses to generate TLS keys.

To avoid modulo bias (a subtle statistical flaw in naive implementations), Vault uses rejection sampling:

    const limit = Math.floor(0x100000000 / max) * max;
    do { crypto.getRandomValues(arr); x = arr[0]; } while (x >= limit);
    return x % max;

This guarantees every value in [0, max) has exactly equal probability.

Random password generation:
1. Pick one character from each enabled set (guarantees at least one of each)
2. Fill the rest from the combined pool
3. Fisher-Yates shuffle using secureRandomInt

Passphrase generation:
1. Pick N words at random from the 400-word list
2. Join with "·" for display, "-" for copy

The entropy math

Entropy is calculated the same way information theorists do it:

    H = log2(N) * L

Where N is the size of the character pool and L is the length.

For passphrases:
    H = log2(400) * N

For PINs:
    H = log2(10) * N

Crack time comes from dividing the average case (half the search space) by the assumed attacker rate:

    seconds = (2^H / 2) / 10^12

The check analysis

The Check mode does more than just measure length. It runs a series of heuristics:

1. Length check — under 8 is flagged red
2. Character variety — counts how many of {lower, upper, digit, symbol} are present
3. Common password check — over 100 of the most-used passwords
4. Repeated characters — detects aaa, 111, etc.
5. Sequences — detects 1234, abcd, and similar
6. Keyboard patterns — detects qwerty, asdfg, zxcvb, etc.
7. Dictionary words — flags any 4+ letter words from the same 400-word list
8. Years — flags 19xx and 20xx
9. Leet speak — flags common substitutions like p@ss, adm1n, r00t

Each finding is shown with a clear explanation. The score is the entropy value, capped at 100, with penalties for the most severe findings.


KEYBOARD SHORTCUTS

Shortcut  Action
R         Regenerate the current password
C         Copy the current password
Esc       Close any open prompt

Note: shortcuts only fire when you're not typing in an input field.


PROJECT STRUCTURE

    vault/
    ├── index.html      The entire app — generator, checker, UI, all of it
    ├── README.md       This file
    ├── LICENSE         MIT
    └── .gitignore

There is no package.json, no node_modules, no build step.


RUNNING LOCALLY

Just open the file:

    open index.html          (macOS)
    start index.html         (Windows)
    xdg-open index.html      (Linux)

Or serve it with any static server:

    python3 -m http.server 8000
    then open http://localhost:8000

Both work. The clipboard API behaves slightly differently on file:// URLs in some browsers, so serving is recommended if you use the copy buttons heavily.


DEPLOYING TO GITHUB PAGES

1. Rename the app file to index.html.
2. Push the repo to GitHub.
3. Go to Settings, then Pages.
4. Set Source to Deploy from a branch.
5. Choose branch main and folder /root.
6. Save.

After about 30 seconds, your site is live.

Works the same on Netlify, Vercel, Cloudflare Pages, or any static host.


BROWSER SUPPORT

Vault uses modern browser features:

- Chrome / Edge 90+
- Firefox 88+
- Safari 15+
- Any Chromium-based browser (Brave, Arc, Opera)

Required APIs:
- crypto.getRandomValues — universally supported
- navigator.clipboard — supported in all modern browsers; falls back to execCommand
- localStorage — universally supported


SECURITY NOTES

What Vault does well

- Uses a real CSPRNG, not Math.random()
- No modulo bias
- Runs entirely client-side
- No network requests after initial page load
- No accounts, no tracking, no analytics
- Open source — you can read every line

What Vault does not do

- It does not store your passwords. If you close the tab, they're gone.
- It does not check whether your password has been leaked in a data breach (that would require a network request).
- It does not integrate with your password manager. Copy-paste manually.
- It cannot protect you from keyloggers, screen recorders, or malware on your own device.
- It cannot stop you from using the same password everywhere.

Best practices

- Use a password manager (Bitwarden, 1Password, KeePassXC)
- Let the manager generate and store your passwords
- Use Vault when you need a quick password for a one-off situation
- Never reuse passwords across sites
- Enable two-factor authentication everywhere it's offered


FAQ

Is Vault really offline?

Yes. After the page loads, no network requests are made. You can verify this by opening DevTools, going to the Network tab, and using the app — nothing appears.

Where is my password stored?

Nowhere. It lives in memory only, in the browser tab. Close the tab and it's gone.

Can someone intercept my password?

Not through Vault. But if your device has malware or a keylogger, that's outside the tool's control. On a compromised machine, nothing is safe.

Why does the Check tab exist if I can just generate a strong password?

Because sometimes you have an existing password you want to evaluate. Maybe it's one you've been using for years. Maybe it's one a website generated for you and you're not sure about. Check tells you the truth.

Does Vault send anything to a "pwned passwords" API?

No. That would break the offline guarantee. If you want breach checking, use Have I Been Pwned directly.

Is this safe to use for my bank password?

Yes, as long as your device is clean. Vault generates stronger passwords than most humans invent. But for anything truly critical, use a hardware key (YubiKey) and 2FA in addition to a strong password.

Why does the app say "longer than the universe" for some passwords?

That's just how the math works out. A 128-bit password has 2^128 possible values. At 10^12 guesses/sec, that's 5.4 * 10^24 years. The universe is about 1.4 * 10^10 years old. So it's not a joke — it's literally true.

Why does my 8-character password show "weak"?

Because it is. 8 characters with all sets on is about 52 bits. That's crackable by a determined attacker with modest hardware in a few hours. The old "8 characters is enough" advice is from 2005 and it's wrong now.


CONTRIBUTING

Contributions are welcome.

1. Fork the repo.
2. Make a change. Bug fixes, new features, doc improvements — anything.
3. Test it. Open the file in a browser. Verify the randomness is still secure.
4. Open a pull request with a clear description.

Guidelines:

- Keep the app a single HTML file.
- No dependencies. No CDN, no framework, no bundler.
- All randomness must come from crypto.getRandomValues. Never Math.random.
- Match the existing code style: two-space indent, single quotes, semicolons.
- If you add a feature, document it in the built-in guide and this README.

If you spot a security issue, please open an issue rather than a public PR.


CREDITS

Author

nardjay — creator and maintainer

Built with

- Vanilla JavaScript — no framework, no bundler, no build step
- crypto.getRandomValues — the browser's built-in CSPRNG
- The 400-word list — curated for memorability, not scraped from a dictionary
- localStorage — for persisting settings (never passwords)

Inspired by

- Correct Horse Battery Staple (XKCD 936) — the original case for passphrases
- Bitwarden, 1Password, KeePassXC — showing what good password hygiene looks like
- Have I Been Pwned — proving that breach checking matters
- The EFF's Diceware wordlist — a model for what a passphrase wordlist should be

Thanks

- To the web platform for shipping crypto.getRandomValues as a first-class API
- To everyone who has ever used "password123" and learned the hard way
- To the open-source community for the culture that made this project possible

Third-party assets

None. No fonts, no icon packs, no CSS frameworks, no libraries, no CDN scripts.

The wordlist is 400 words, hand-picked and deduplicated. Everything else is inline SVG and Unicode.


LICENSE

Released under the MIT License.

You're free to use, modify, and distribute Vault, including for commercial purposes. Attribution is appreciated but not required. See the LICENSE file for the full text.

Remember: a password is only as safe as the device you generate it on.
