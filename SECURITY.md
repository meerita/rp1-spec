# Security Policy

## Supported revisions

RP-1 Spec has no published revision. Report a problem against the current
state of the `master` branch.

## Reporting

Send a report to **meerita@icloud.com**.

Do not open a public issue. Do not describe the problem publicly before a
correction exists.

Include what you can:

- The section, and the exact clause.
- What an implementation built from that clause does wrong.
- What it lets an attacker do.
- A byte sequence or an exchange that reaches the problem, if one exists.
- Whether a known implementation is affected.
- A suggested correction, if you have one.

## What to expect

You get an acknowledgement. You get an assessment of whether the problem
is confirmed and of what it affects. You get told when a correction
lands, and in which revision.

A confirmed problem is corrected before it is described publicly. The
report credits you unless you ask otherwise.

## Scope

This repository defines a contract. A security problem here is a defect
in the contract, not in a program.

In scope:

- A clause that an implementation cannot satisfy safely.
- A length, count, or size an implementation is told to act on before it
  is told to check it.
- A field whose bound is never stated, so a receiver cannot size its
  work.
- A missing receiver rule that leaves implementers to invent one, where
  some inventions are unsafe.
- A validation order that permits work or allocation before the input is
  known to be legal.
- An extension rule that lets an attacker reach undefined behavior with
  input the contract permits.
- A failure whose scope is unstated, so a receiver cannot tell whether
  the connection is still trustworthy.

Out of scope here:

- A defect in the RP-1 server. Report it through that project.
- A defect in an SDK. Report it through that SDK's project.
- An implementation that does not follow a clause that is itself correct.

An implementation defect that turns out to come from an ambiguous clause
is in scope. If you are not sure which you have, report it here and say
so.
