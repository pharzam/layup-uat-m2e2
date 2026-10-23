head: a14ecc47c4792eb6106cda07f970abacc1af373e
verdict: not mergeable, findings recorded
findings:
- docs/adr/0012-parse-flags-by-hand.md:36 R13: Rule 4 makes every remaining argument a filename, so tally file -l opens a file named -l, while lines 50-53 explicitly leave that case undecided pending an idea-owner answer. SPEC-0002 lines 61-68 repeats the conflict and additionally classifies -- before a filename as a usage error while declaring its meaning unresolved. Exclude the explicitly unresolved cases from the operative parser rules in both documents, or obtain facts resolving them; an implementer currently has contradictory instructions about whether to open, reject or defer these arguments.
