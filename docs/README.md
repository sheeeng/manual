# Commands

While both commands produce the same HTML documentation, they represent two different "eras" of the Nix command-line interface.

## NixOS Manual

### Verbatim Commands

#### Classic Version

```shell
nix-build \
  --expr 'with import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-unstable.tar.gz") {};
    (import (path + "/nixos/lib/eval-config.nix") {
      modules = [ ({ pkgs, ... }: { system.stateVersion = "25.11"; }) ];
    }).config.system.build.manual.manualHTML
  '
```

#### Modern Version

```shell
nix build --impure --expr '
  let
    pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-unstable.tar.gz") {};
    nixos = import (pkgs.path + "/nixos/lib/eval-config.nix") {
      modules = [ ({ ... }: { system.stateVersion = "25.11"; }) ];
    };
  in
    nixos.config.system.build.manual.manualHTML
'
```

### Simpler Version

```shell
nix-build '<nixpkgs/nixos>' --attr options.manualHTML
```

The simpler version of the command is not reproducible. It depends on whatever your local channel points to, which can change over time. Different machines might build different versions. It relies heavily on local system configuration rather than being self-contained. It is assuming that the `NIX_PATH` environment variable is configured. It also requires "channel," which is a name for the latest "verified" git commits in Nixpkgs.

The angle brackets `<nixpkgs>` perform a lookup in your `NIX_PATH` environment variable, typically set when you subscribe to a Nix channel.

```shell
nix-channel --add https://nixos.org/channels/nixos-unstable nixpkgs
nix-channel --update
```

To see the actual path that `<nixpkgs>` points to:

```shell
nix-instantiate --find-file nixpkgs
```

To see the actual path that `<nixpkgs/nixos>` points to:

```shell
nix-instantiate --find-file nixpkgs/nixos
```

---

## Nixpkgs Manual

The nixpkgs manual documents the packages, functions, and libraries available in nixpkgs.

### Modern Version

```shell
nix build --impure --expr '
  let
    pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-unstable.tar.gz") {};
  in
    pkgs.nixos.htmlDocs.nixpkgsManual
'
```

### Simpler Version

```shell
nix-build '<nixpkgs>' --attr nixos.htmlDocs.nixpkgsManual
```

The simpler version has the same limitations as described above for the NixOS manual. For reproducible builds, use the modern version with `fetchTarball`.

---

## Structural Comparison

| Feature          | `nix-build` (Classic)   | `nix build` (Modern)                   |
| :--------------- | :---------------------- | :------------------------------------- |
| **CLI Status**   | Stable / Legacy         | Experimental / New                     |
| **Strictness**   | Impure by default       | Pure by default (requires `--impure`)  |
| **Logic Style**  | Uses `with` (Implicit)  | Uses `let...in` (Explicit)             |
| **Fetch Method** | Standard `fetchTarball` | Requires `--impure` for `fetchTarball` |
| **Output Link**  | Creates `./result`      | Creates `./result`                     |

---

## Key Differences Explained

### CLI Generation

- **`nix-build`**: Part of the original Nix toolset. It is highly compatible and does not require experimental features to be enabled in `nix.conf`.
- **`nix build`**: Part of the new Nix 2.0+ CLI. It is designed to be more consistent and works natively with Flakes.

### Implicit vs. Explicit Scoping

- **Implicit (`with import ...`)**: The classic command uses `with`, which dumps all variables from Nixpkgs into the scope. It is shorter to type but can make it unclear where a variable like `path` is coming from.
- **Explicit (`let pkgs = ... in`)**: The modern command uses a `let` block to define variables. This is considered best practice because it makes the code easier to read and debug.

### Purity and Reproducibility

The modern `nix build` command enforces "purity." Because the URL `nixos-unstable.tar.gz` points to a file that changes every day, Nix considers it an **impure** input. You must explicitly add the `--impure` flag to tell Nix you acknowledge this input is not fixed.

### Attribute Path Accuracy

In the modern example, we use `pkgs.path` instead of a naked `path`. This ensures that the NixOS evaluation logic (`eval-config.nix`) is pulled exactly from the same source code as the packages being used, preventing version mismatches.

The simpler version requires a NixOS system or properly configured `NIX_PATH`. On non-NixOS systems (like macOS), use the commands shown above with `fetchTarball` instead.
