# koushi-matrix-bin (AUR packaging)

Source of truth for the [`koushi-matrix-bin`](https://aur.archlinux.org/packages/koushi-matrix-bin)
AUR package, which repackages the official `Koushi-linux-x64.deb` release asset
from [shinaoka/koushi-matrix](https://github.com/shinaoka/koushi-matrix).

## Automation

[`update-aur.yml`](.github/workflows/update-aur.yml) runs daily. It reads the
newest upstream release, and when the version differs from `PKGBUILD` it takes
the SHA-256 from the release's own `.sha256` file, bumps `pkgver`, resets
`pkgrel`, builds the package once to prove the source resolves, and pushes the
result to the AUR.

Nothing happens when upstream has not released, when the release carries no
Linux `.deb`, or when the checksum file is malformed.

The workflow needs no attention: it also commits a monthly stamp file, so
GitHub never suspends the schedule for the 60-day inactivity rule. A failing
run emails the repository owner. The Actions tab still offers **Run workflow**
for an immediate check after an upstream release.

## Manual release

```bash
git clone ssh://aur@aur.archlinux.org/koushi-matrix-bin.git
cd koushi-matrix-bin
# edit pkgver / pkgrel
updpkgsums
makepkg --printsrcinfo > .SRCINFO
makepkg -f            # verify it builds
git commit -am "Update to <version>" && git push
```

## Packaging notes

- `-bin` package: it unpacks the upstream deb rather than building from source,
  which would otherwise need the Rust toolchain, Node, and the vendored
  matrix-rust-sdk submodule for every user.
- `depends` mirrors what the shipped binary actually links against; the
  `libsecret` dependency backs credential storage through the freedesktop
  Secret Service.
- `options=('!strip' '!debug')` keeps the upstream binary byte-identical.
