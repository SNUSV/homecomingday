# CLAUDE.md

## Project

Static one-page site for **SNUSV 30주년 홈커밍데이** (Seoul National University 벤처동아리 30th anniversary). Single `index.html` (~4 MB, all CSS/JS/most images inlined as base64) plus a `logos/` folder of sponsor/partner images. No build tooling. Deployed to GitHub Pages at https://snusv.github.io/homecomingday/.

## Gotcha: Korean filenames must be NFC, not NFD

macOS stores Korean filenames in **NFD** (decomposed jamo, e.g. `모` = ㅁ+ㅗ, bytes `e1 84 86 e1 85 a9`) by default. APFS does normalization-insensitive lookups, so requests for NFC-composed names (`모` = `eb aa a8`) resolve locally. **GitHub Pages serves from Linux with byte-exact matching, so NFC requests 404 against NFD files** — logos with Korean names appear broken in production but fine locally.

The HTML in this repo references filenames in NFC, so files on disk must also be NFC.

**Why:** Already hit this once — the six Korean-named logos in `logos/` (모닝펀치, 세사미, 셀러로그, 엘리트잡, 인블로그, 인테이크, plus 벤처기업협회) all 404'd on Pages until renamed to NFC.

**How to apply:** Any time a new file with non-ASCII (especially Korean) characters is added to the repo on macOS — whether by drag-and-drop, download, unzip, or `mv` — normalize it to NFC before committing. To check/fix the whole repo:

```sh
python3 -c "
import os, unicodedata
for root, dirs, files in os.walk('.'):
    if '/.git' in root: continue
    for f in files:
        nfc = unicodedata.normalize('NFC', f)
        if nfc != f:
            os.rename(os.path.join(root, f), os.path.join(root, nfc))
            print(f'renamed: {f!r} -> {nfc!r}')
"
```

To verify a single filename is NFC, `ls logos/ | xxd` and check that Hangul bytes are in the `ea`–`ed` range (precomposed syllables) rather than starting with `e1 84`/`e1 85`/`e1 86` (decomposed jamo).
