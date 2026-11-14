# FP Square Root

This is the starter/specification repository for this course's project: an IEEE-754 **single-precision floating-point square root** unit. Groups 17 to 19 work on this project. Each group **creates its own repository from this template** (**Use this template** → **Create a new repository**) and builds its design there.

Maintainer: COD TA's of '27 graduating batch.

## How this repository works

- This repository defines the **module contract**. It does not contain any RTL implementation.
- **One repository per group.** One member creates it from this template and adds the other members as collaborators.
- Groups do not submit pull requests back to this repository. It is the shared specification, not a shared codebase.
- Module, port and parameter names are up to you. The **port count, directions, widths and behavior** must match the specification, and you tell the CI how your names map to it in `rnr-stage.yaml`. See the assignment handout.

## Module

Click on the module name to see its folder and specification:

- [FP_Sqrt](FP_Sqrt/)

## Tools

- **Vivado is the official/recommended development environment** for this course.
- The CI synthesizes your RTL with **Yosys 0.33**, which cannot parse SystemVerilog assignment patterns such as `'{default: 0}`. Use a `for` loop to clear arrays. File paths in `rnr-stage.yaml` must not contain spaces.

## What the CI checks

The CI builds your design in a sandbox and compares its behaviour against a trusted reference. The details (ports, handshake, what is covered) are in [FP_Sqrt/README.md](FP_Sqrt/README.md). The check runs on the next poll after you push, which can take several hours. Compliance tests, your architecture description, testbench and waveforms are part of the course deliverables but are **not** checked by the CI.

## Submission Workflow

1. Create your group's repository from this template and add your group members.
2. Install the **`risc-n-roll-ci`** GitHub App on **that exact repository** (otherwise the CI never sees it).
3. Write your design under `FP_Sqrt/` (or anywhere you like, then update `rnr-stage.yaml`), commit and push.
4. Submit your repository URL (one per group) through the course Google Form. The assignment handout explains how to configure and verify your repository.
