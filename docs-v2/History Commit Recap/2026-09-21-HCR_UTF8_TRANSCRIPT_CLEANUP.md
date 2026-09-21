# Commit Recap: Preserve Accented Characters During Transcript Cleanup

Status: Active
Last updated: 2026-09-21 23:01
Commit: This commit
Base commit: b477065e
Related files: `src/core/ngram_loop_fix.h`, `tests/test-ngram-loop-fix.cpp`, `.github/workflows/build-windows-vulkan-manual.yml`

## Summary

Make shared transcript cleanup preserve accented and other non-ASCII characters regardless of the Windows language setting.

## Why This Work Happened

A real French recording produced valid `Voilà donc normalement|` before cleanup. The returned text lost byte A0 from the two-byte encoding of `à`, causing the application to reject the damaged text and stop transcription. Windows treated that individual byte as whitespace under its English/1252 character setting.

## What Changed

- Replace locale-dependent byte classification with the six ordinary ASCII whitespace characters. The existing word-splitting function is smaller; repetition removal is unchanged.
- Add a Windows regression using the captured French text, a Chinese character containing the same byte, and ordinary whitespace checks.
- Run only the existing text-cleanup test executable in the manual Windows build before compiling the DLL. Example targets are configured because the existing test definitions reference them; they are not built or packaged.

## How This Is Supposed To Help

Valid model text remains valid after shared cleanup, without hiding errors or changing the model's words in Python.

## Risk And Behavior Changes

The splitter is shared by several native speech engines. Ordinary whitespace normalization and repeated-phrase removal remain in place. Recognition quality, GPU cost, and streaming delivery are outside this change.

## Verification

The previous DLL's corruption was reproduced with saved speech and observed before and after native cleanup. Source review and Git whitespace checks passed. New Windows tests and DLL compilation will run through the manual GitHub workflow after this commit; no claim of passing them or application acceptance is made here.
