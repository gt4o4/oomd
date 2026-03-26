# RPM Packaging for oomd

This directory contains the RPM spec file and patches for building oomd as an
RPM package (for Fedora, RHEL, CentOS, etc.).

## Origin

The packaging was imported from the official Fedora package source:

- **Source:** https://src.fedoraproject.org/rpms/oomd (rawhide branch)
- **Import date:** 2026-03-26
- **Fedora version at import:** 0.5.0-17

The spec has been adapted for the current HEAD of this repository:

- **Patches 0–2** from the Fedora spec (commits `076af42`, `3989e16`,
  `83a6742`) have been dropped because they are already merged upstream.
- **Patch 0** (in our spec) disables a `CgroupContextTest.DataLifeCycle`
  assertion that fails on kernel ≥ 6.6.9 due to `fdopendir` behavior changes.
  This is the Fedora-local patch `oomd-disable-datalifecycle-children-test.patch`.

## Files

| File | Description |
|------|-------------|
| `oomd.spec` | RPM spec file (adapted from Fedora rawhide) |
| `oomd-disable-datalifecycle-children-test.patch` | Patch to disable flaky test on newer kernels |

## Building an RPM

### Prerequisites

```bash
sudo dnf install rpm-build rpmdevtools meson gcc-c++ jsoncpp-devel \
    systemd-devel gmock-devel gtest-devel
```

### Using rpmbuild

```bash
# Set up the RPM build tree
rpmdev-setuptree

# Copy spec and patches
cp rpm/oomd.spec ~/rpmbuild/SPECS/
cp rpm/*.patch ~/rpmbuild/SOURCES/

# Create a source tarball from the current tree
git archive --format=tar.gz --prefix=oomd-0.5.0^118.g88c28fd/ \
    -o ~/rpmbuild/SOURCES/oomd-0.5.0^118.g88c28fd.tar.gz HEAD

# Build
rpmbuild -ba ~/rpmbuild/SPECS/oomd.spec
```

### Using mock (recommended for clean builds)

```bash
# Create the SRPM first
rpmbuild -bs ~/rpmbuild/SPECS/oomd.spec

# Build in a clean chroot
mock ~/rpmbuild/SRPMS/oomd-*.src.rpm
```

### Disabling tests

To build without running tests:

```bash
rpmbuild -ba ~/rpmbuild/SPECS/oomd.spec --without tests
```

## Keeping in sync with Fedora

To check for updates to the Fedora spec:

```bash
# Clone the Fedora package repo
fedpkg clone oomd fedora-oomd
cd fedora-oomd
git log --oneline -10
```

Or browse directly: https://src.fedoraproject.org/rpms/oomd/commits/rawhide
