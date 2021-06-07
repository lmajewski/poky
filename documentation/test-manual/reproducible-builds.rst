.. SPDX-License-Identifier: CC-BY-SA-2.0-UK

*******************
Reproducible Builds
*******************

================
How we define it
================

The Yocto Project defines reproducibility as where a given input build configuration will give the same binary output regardless of when it is built (now or in 5 years time), regardless of the path on the filesystem the build is run in and regardless of the distro and tools on the underlying host system the build is running on.

==============
Why it matters
==============

The project aligns with the Reproducibile Builds project (https://reproducible-builds.org/) and they have information about why it matters. In addition to being able to validate for security issues being introduced which they talk about at length, from a Yocto Project perspective, it is hugely important that our builds are deterministic. We expect that when you build a given input set of metadata, you get consistent output. This has always been a key focus but is now true down to the binary level including timestamps.

For example, if you find at some point in the future life of a product that you need to rebuild to add a security fix, only the components that have changed should change at the binary level leading to much easier and clearer bounds on where validation is needed.

This also gives an additional benefit to the project builds themselves, our hash equivalence for sstate object reuse works much more effecitvely when the binary outputis unchanging.

===================
How we implement it
===================

There are many different aspects to build reproducibility but some particular things we do within the build system to ensure reproducibility include:

* Adding mappings to the compiler options to ensure debug filepaths are mapped to consistent target compatible paths. This is done through the ``DEBUG_PREFIX_MAP`` variable which sets the maco-prefix-map and debug-prefix-map compiler options correctly to mape to target paths.
* Being explict about recipe dependencies and their configuration (no floating configure options or host dependencies creeping in). In particular this means making sure ``PACKAGECONFIG`` coverage covers configure options which may otherwise try and auto-detect host dependencies.
* Using recipe specific sysroots to isolate recipes so they only see their dependencies. These are visible as "recipe-sysroot" and "recipe-sysroot-native" directories within ``WORKDIR`` of a given recipe and are poulated only with the dependencies a recipe has.
* Build images from a reduced package set of only packages from recipes the image depends upon.
* Filtering the tools availble from the host's PATH to only a specific set of tools, set using the ``HOSTTOOLS`` variable.

=========================================
Can we prove the project is reproducible?
=========================================

Yes, we can prove it and we now regularlly test this on the autobuilder. At the time of writing, OE-Core is 100% reproducible for all it's recipes (i.e. world builds) apart from go-lang and ruby's docs package. go-lang has some fundamental problems with reproducibility as it currently always depends upon the paths it is build in unfortunately.

To run our automated selftest as we use in our CI on the autobuilder you can run::

 oe-selftest -r reproducible.ReproducibleTests.test_reproducible_builds

This defaults to including a world build so if other layers are added, it would also run the tests for recipes in the additional layers. The first build will be run using sstate if available, the second build expliclty disables sstate and builds on the specific host the build is running on. This means we can test reproduibility builds between different host distributions over time on the autobuilder.

If ``OEQA_DEBUGGING_SAVED_OUTPUT`` is set, any differing packages will be saved here. The test is also able to run diffoscope on the output and generate html files showing the differences between the packages which can aid debugging. On the autobuilder these appear under the url https://autobuilder.yocto.io/pub/repro-fail/ in the form oe-reproducible + <date> + <random ID>, e.g. "oe-reproducible-20200202-1lm8o1th".

The project's current reproducibility status can be seen at: https://www.yoctoproject.org/reproducible-build-results/

===============================
Can I test my layer or recipes?
===============================

Yes, see the selftest in oe-selftest at meta/lib/oeqa/selftest/cases/reproducible.py which can be subclassed for specific use cases.





