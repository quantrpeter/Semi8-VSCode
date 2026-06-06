![](banner.png)

# Semi8 Assembly for VS Code

Syntax highlighting for the **Semi8** (also known as Open8) 8-bit microcontroller assembly language.

![](/img/Screenshot20260606.png)

## Features

- Syntax highlighting for Semi8/Open8 assembly (`.s`, `.S`, `.s8`, `.semi8`, etc.)
- Recognizes all instructions from the Semi8 instruction set (including implemented subset and documented extensions)
- Highlights registers (`R0`–`R31` and index `X`/`Y`/`Z` variants), labels, numbers (decimal, hex `0x`, binary `0b`), comments (`;` and `//`), and common assembler directives (`.equ`, `ORG`, `DB`, etc.)
- Comment toggling with `;`

## Usage

1. Open any `.s` (or supported extension) file containing Semi8 assembly.
2. If the language is not auto-detected, use the command palette (`Cmd+Shift+P` / `Ctrl+Shift+P`) and select **"Change Language Mode"** → **"Semi8 ASM"**.

Alternatively, add to your VS Code `settings.json`:

```json
"files.associations": {
  "*.s": "semi8-asm"
}
```

**Tip:** To debug highlighting, run the **Developer: Inspect Editor Tokens and Scopes** command (from Command Palette) and hover over tokens in a Semi8 file. This shows the scopes applied by the TextMate grammar.

## Supported Instructions (highlights)

Includes (case-insensitive):
`ADC ADD AND ANDI ASR BLD BREAK BRCC BRCS BREQ BRGE BRLT BRNE BST CBR CBI CALL CLR COM CPC CP DEC EOR INC IN JMP LD LDD LDI LDS LPM LSL LSR MOV MOVW MUL NEG NOP OR ORI OUT POP PUSH RET RETI RJMP ROL ROR SBC SBI SBR SEI CLI SLEEP SPM ST STD STS SUB SUBI SWAP TST WDR` and pseudo-ops like `CLR LSL ROL TST SER`.

See the [Semi8 instruction reference](https://github.com/semiblock/semi8/blob/main/doc/instruction.md) and [program example](https://github.com/semiblock/semi8/blob/main/program.s) in the main Semi8 repository.

Also supports common assembler directives (case-insensitive, with or without dot prefix): `EQU SET DB DW DL DS ORG SECTION MACRO ENDM INCLUDE IF ELSE ENDIF DEFINE`.

## Development & Debugging

### Running the extension from source

1. Open the `Semi8-VSCode` folder directly in VS Code (recommended over a multi-root workspace for extension development).
2. Press `F5`, or go to the **Run and Debug** view and select **"Run Extension"**.
   - This starts an **Extension Development Host** — a separate VS Code window that loads the extension directly from your source files.
3. In the Extension Development Host:
   - Open a test file, for example the official example:
     `/Users/peter/workspace/Semi8/program.s`
   - Or any file with one of the registered extensions (`.s`, `.S`, `.s8`, `.semi8`, `.asm8`, `.o8`).
   - If the language is not automatically detected, open the Command Palette and run **Change Language Mode → Semi8 ASM**.

A `.vscode/launch.json` is included so F5 works out of the box.

### Debugging syntax highlighting (the main way to debug this extension)

Because this is a **pure declarative language extension** (no `main.ts`, no activation code), you don't debug JavaScript — you debug the **TextMate grammar**.

Use this command (the single most useful tool):

1. In the Extension Development Host, open a Semi8 `.s` file.
2. Open the Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`).
3. Run **Developer: Inspect Editor Tokens and Scopes**.
4. Click or move the cursor over any token.

You will see:
- The scopes that were applied (e.g. `keyword.control.instruction.semi8-asm`, `variable.language.register.semi8-asm`, `entity.name.function.label.semi8-asm`, `comment.line.semi8-asm`).
- Which rule in `syntaxes/semi8.tmLanguage.json` produced the match.

This is how you verify and iterate on the grammar.

### Applying changes while developing

After you edit any of these files:
- `syntaxes/semi8.tmLanguage.json`
- `language-configuration.json`
- `package.json`

Do the following in the **Extension Development Host** window:
- Run **Developer: Reload Window** from the Command Palette, or
- Press the reload button in the title bar of the dev host.

Then re-open your test file or re-run the token inspector.

### Testing language configuration features

- **Comment toggling**: Select one or more lines and press `Ctrl+/` (`Cmd+/` on Mac). It should insert `;` comments.
- **Brackets and auto-closing**: Defined in `language-configuration.json`.
- **Word selection**: The `wordPattern` affects double-clicking on labels and registers.

### Testing a packaged `.vsix` (instead of source)

```bash
npx @vscode/vsce package --no-git-tag-version
```

Then, in a **regular** VS Code window (not the Extension Development Host):
- Go to the Extensions view.
- Click the `...` menu → **Install from VSIX...**
- Choose the generated `semi8-vscode-*.vsix` file.

Close that window when you want to go back to source-based development with F5.

### Important note about `.s` files

The extension `.s` is extremely common (AVR, ARM, x86 gas, etc.). VS Code may default to another language. You can force it with:

```json
// settings.json
"files.associations": {
  "*.s": "semi8-asm",
  "*.S": "semi8-asm"
}
```

Or use **Change Language Mode** per file.

## Packaging

```bash
npm install -g @vscode/vsce
vsce package
```

This produces a `.vsix` file you can install manually via **Extensions: Install from VSIX...** or `code --install-extension semi8-asm-*.vsix`.

## Publishing to the VS Code Marketplace

### Prerequisites

1. **Create / own a publisher**
   - Go to https://marketplace.visualstudio.com/manage
   - Log in with a Microsoft account.
   - Create a publisher (or select an existing one).  
     The `publisher` field in `package.json` is currently set to `"semiblock"`. You must own the `semiblock` publisher, or change the field to your own publisher ID.

2. **Create a Personal Access Token (PAT)**
   - In Azure DevOps (https://dev.azure.com), go to **User settings → Personal access tokens → New Token**.
   - Set:
     - Organization: **All accessible organizations**
     - Scopes → Custom → **Marketplace (Manage)**
   - Copy the token (you will only see it once).

3. **Install vsce**
   ```bash
   npm install -g @vscode/vsce
   ```

4. **(Recommended) Add a proper icon**
   - Create a square PNG icon (minimum 128×128, ideally 256×256) — e.g. `icon.png`.
   - Do **not** use the wide `banner.png`.
   - Add to `package.json`:
     ```json
     "icon": "icon.png"
     ```
   - Re-package/publish after adding.

### Publish steps

```bash
# 1. Login with your publisher (stores the token securely), skip this step if you already ran it
vsce login hongkongprogrammingsociety

# 2. (Optional but recommended) Test by packaging first
vsce package

# 3. Publish (bumps patch version by default, or use `minor` / `major` / explicit version)
vsce publish
# or with the token directly:
# vsce publish -p <your-pat-here>

# To publish a specific version increment:
# vsce publish minor
```

- `vsce publish` will:
  - Update the version in `package.json` (if you pass `minor` etc.).
  - Create a git tag/commit if you're in a git repo (optional).
  - Upload to the Marketplace.

After publishing, the extension will appear at:
`https://marketplace.visualstudio.com/items?itemName=hongkongprogrammingsociety.semi8-asm`

Users can then install it directly from the Extensions view in VS Code.

### Tips & common issues

- **403 / 401 errors**: Make sure the PAT has **Marketplace (Manage)** and **All accessible organizations**.
- **Icon requirements**: Must be a PNG (no SVG). 128×128 minimum.
- **Links & images**: All image URLs in README must be https. Relative links are rewritten automatically by vsce when a `repository` field points to a public GitHub repo (already set).
- **Keywords limit**: Max 30 keywords (we are well under).
- **First publish**: The extension name + publisher combination must be unique.
- For CI/CD (GitHub Actions, Azure Pipelines): Prefer Microsoft Entra ID / workload identity federation over long-lived PATs (see official docs).

See the official guide: https://code.visualstudio.com/api/working-with-extensions/publishing-extension

## Related

- [Semi8 Project](https://github.com/semiblock/semi8)
- [SemiBlock](https://www.semiblock.ai)
