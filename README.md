# Zipover Desktop releases

Copy text, images, files and folders between your computers over an encrypted local connection.

- [Website and downloads](https://zipover.app/download/) · [中文下载与安装说明](https://zipover.app/cn/download/)
- [Release history](https://github.com/harrisonwang/zipover-releases/releases)
- [Issues and feedback](https://github.com/harrisonwang/zipover-releases/issues)
- [Free and open-source CLI](https://github.com/harrisonwang/zipover)

Desktop offers a 30-day offline trial, starting when you explicitly choose “开始 30 天免费试用” in the app. No account, payment details or license key are needed for the trial. Expiry pauses synchronization and retains pairings. Purchases are not open yet.

The website identifies the current release and explains supported architectures and first-launch requirements. Release candidates are marked as prereleases. The stable [1.0.0 release](https://github.com/harrisonwang/zipover-releases/releases/tag/v1.0.0) is available. Current macOS packages are ad-hoc signed without Apple notarization, and Windows packages are unsigned.

Each release includes its applicable `LICENSE`, third-party notices and `SHA256SUMS.txt`. Use the license shipped with the downloaded version. Keep private license keys and sensitive clipboard contents out of public issues.

This repository distributes Desktop binaries and receives feedback. Desktop source is maintained separately; CLI/core source remains public.

## R2 download mirror

Published Desktop releases are mirrored by `.github/workflows/mirror-to-r2.yml`.
A GitHub App publishes releases in this repository, so `release: published` triggers
the mirror. The workflow can also be run manually with an existing published tag
to recover a failed upload or mirror an older release.

Configure these repository secrets: `R2_ACCOUNT_ID`, `R2_BUCKET`,
`R2_ACCESS_KEY_ID`, and `R2_SECRET_ACCESS_KEY`. The R2 S3 credentials need object
read/write access only to the dedicated download bucket. Set the repository
variable `R2_PUBLIC_BASE_URL` to the bucket's HTTPS public origin, such as a
connected custom download domain. The workflow requires this origin and verifies
all public downloads before reporting success; an S3 API endpoint is not a public
download origin.

Each release keeps all ten original GitHub filenames under
`desktop/releases/<tag>/`: four installers, four third-party notices, `LICENSE`,
and `SHA256SUMS.txt`. For example, the Apple Silicon package for `v1.0.0` has key
`desktop/releases/v1.0.0/zipover-gui_v1.0.0_aarch64-apple-darwin.dmg`.
There is no `latest` alias, so a release candidate or an old manual rerun cannot
move a stable download to another version.

Before any upload, the script checks the exact asset set and every published
SHA-256 checksum, then compares any existing R2 objects by downloading and hashing
their contents. An identical rerun is safe; different content at an existing key
fails. New objects use atomic `If-None-Match: *` writes, Content-MD5, and a one-year
immutable cache policy. Every upload is read back and SHA-256 verified, with the
checksum manifest uploaded last. A partial failure can be rerun without replacing
already verified files. Do not replace files in an existing version; publish a
new version instead.

Install `scripts/requirements-mirror.txt` in a Python virtual environment, then
run `python -m unittest discover -s scripts -p 'test_*.py' -v` to test validation,
partial recovery, replay, immutable writes, and concurrent-upload handling.

Implementation references: [R2 conditional writes](https://developers.cloudflare.com/r2/api/s3/extensions/)
and [GitHub workflow triggering](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow).
