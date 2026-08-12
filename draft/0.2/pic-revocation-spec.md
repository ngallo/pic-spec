---
title: "PIC Revocation"
abbrev: "PIC Revocation"
docname: pic-revocation-02
category: info
ipr: none
submissiontype: independent
stand_alone: yes
smart_quotes: false
pi: [toc, sortrefs, symrefs]
date: 2026-07-18

author:
  - ins: N. Gallo
    name: Nicola Gallo
    org: Nitro Agility S.r.l.
    email: nicola.gallo@nitroagility.com

normative: {}
informative: {}
---

--- note_Document_Status

**Project:** PIC Protocol
**Project Website:** [www.pic-protocol.org](https://www.pic-protocol.org/)
**Document:** pic-revocation-02
**Version:** 0.2 (Draft)
**Document Status:** Public Draft
**Intended Use:** Informational and Experimental
**Published:** 2026-07-18
**Editor(s):** Nicola Gallo (Nitro Agility S.r.l.)
**Steward:** Nitro Agility S.r.l.
**Source:** [github.com/pic-protocol/pic-spec/draft/0.2/pic-revocation-spec.md](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-revocation-spec.md)

--- note_About_This_Document

> **Public Draft — Not a Standard**
>
> This document is an independently developed specification published as part of the PIC Protocol and maintained by Nitro Agility
> S.r.l. in its role as Specification Steward.
>
> It has not been adopted, endorsed, approved, or published by the IETF, IRTF, IAB, RFC Editor, ISO, IEC, W3C, CNCF, OpenID
> Foundation, or any other standards-development organization, unless a later version explicitly states otherwise.
>
> It is not an RFC, an Internet Standard, or an official work item of any working group or standards body.
>
> This document is published for public review, research, experimentation, implementation feedback, and possible future
> standardization work. It may be revised, replaced, or withdrawn at any time.
>
> Implementers use this draft at their own risk. Any implementation, interoperability statement, or conformance claim applies only
> to the exact document version identified above.
>
> Publication of this draft does not constitute certification, endorsement, security approval, interoperability assurance,
> regulatory approval, or standards-body recognition.
>
> Current project information and published specifications are available at `https://www.pic-protocol.org/`.

--- to_be_removed_note_Work_In_Progress

**Work in Progress — Experimental Design**

The revocation model described in this document is under active development.

This model may change substantially before a later stable revision.

Implementers MUST NOT assume wire compatibility, semantic compatibility, or backward compatibility with future revisions.

Future drafts may introduce breaking changes to terminology, data structures, processing rules, profile bindings, and
interoperability requirements.

The current text is published for design review, experimentation, and implementation feedback.

--- note_Editors

- **Nicola Gallo** (Nitro Agility S.r.l.) Lead Editor
- *Add your name via pull request (individual or organization) — listing is subject to editor approval (see [Contributors](#contributors)).*

--- note_Contributors

- *Add your name via pull request (individual or organization) — listing is subject to editor approval (see [Contributors](#contributors)).*

--- abstract

This document is the **PIC Revocation Specification**, a subordinate specification of the
[PIC Specification](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-spec.md). It defines how authority is revoked within
the Provenance Identity Continuity (PIC) Model.

Expiry bounds how long a continuity state may be accepted; revocation withdraws validity before that bound. PIC revocation preserves the
core model invariants: exactly one predecessor per non-root advancement, Proof of Relationship, authority non-expansion, and no authority
import from unrelated continuities.

For the current PIC Profile 0.2 / PIC-X realization, the native causal revocation coordinate is a stable authenticated **PCA ID** plus a
continuity **position**. Additional authenticated selectors, such as grant, issuer, key, delegate, attestation, or policy selectors, may be
used by a profile but do not replace the causal coordinate when descendants must be invalidated.

--- middle

# Introduction

This section is non-normative. It introduces the revocation model used by this document.

PIC continuity starts from a trusted root PIC Context of Authority, or PCA, and advances through non-root continuity advancements. A
revocation that must affect descendants needs a stable way to name the root authority context and a stable way to name the causal position at
which validity stops.

In PIC Profile 0.2, PIC-X issues settled PIC Continuity JWTs. A workload may propose exactly one advancement in a candidate PIC Continuity
JWT by carrying one `continuity_transition_jwt`. PIC-X validates that candidate and, if it accepts the advancement, issues the next settled
PIC Continuity JWT. Settled Profile 0.2 continuity artifacts do not carry a replayable transition history.

Revocation state remains external and dynamic. Revoking a PCA, position, key, delegate, attestation, grant, issuer, or policy does not
rewrite the root PIC PCA JWT and does not rebuild historical transitions. Verifiers and PIC-X evaluate authenticated revocation state against
the presented continuity state.

## Requirements Notation

The normative sections of this document are Sections 2, 3, 4, and 5. Sections 1 and 6 are non-normative.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" are to be interpreted as described in BCP 14 [[2]](#references) [[3]](#references) when, and only when, they appear in all capitals
within normative text.

# Revocation Coordinates

This section is normative.

## PCA ID

A revocable Profile 0.2 continuity MUST have a stable authenticated **PCA ID**. The PCA ID identifies the root authority context for
revocation purposes.

The PCA ID MUST be bound to the trusted root PIC PCA JWT or to authenticated root metadata covered by the selected profile. It MUST NOT be a
freely chosen unauthenticated value. The selected profile MUST define how the PCA ID is represented, authenticated, and validated.

This document does not define the concrete wire claim carrying the PCA ID. It also does not define the JWT `jti` claim as the PCA ID. A
profile MAY map a concrete claim to the PCA ID only when that mapping is explicitly defined and authenticated by that profile.

The compact signed root PIC PCA JWT is carried by a settled Profile 0.2 PIC Continuity JWT at:

~~~text
context_of_authority.root.pca_jwt
~~~

Its compact-artifact hash is carried at:

~~~text
context_of_authority.root.pca_jwt_hash
~~~

Those fields identify the root artifact. Whether either value is used directly as the PCA ID is profile-defined.

## Position

A revocable Profile 0.2 continuity MUST expose or authenticate a continuity `position`.

The position identifies causal order within the continuity rooted at the PCA ID. The root position and non-root positions MUST be
unambiguous. A non-root advancement MUST have the position required by the selected continuity profile.

The profile MUST reject missing, negative, fractional, non-canonical, overflowing, wrapping, or otherwise invalid positions. A profile MUST
NOT introduce a second mandatory counter solely for revocation when the continuity profile already defines positions.

## Causal Coordinate

The native causal coordinate for current Profile 0.2 revocation is:

~~~text
(PCA ID, position)
~~~

A causal suffix cutoff is:

~~~text
(PCA ID, fromPosition)
~~~

It matches a presented continuity state when:

~~~text
state.PCA_ID == target.PCA_ID
AND state.position >= target.fromPosition
~~~

Whole-continuity revocation is the root-position case. The selected profile MUST define the root position used for that comparison.

Knowledge of the PCA ID, the position, or both MUST NOT authorize revocation. Targeting and revocation authorization are separate.

# Revocation Semantics

This section is normative where BCP 14 language is used.

## Causal Suffix

A causal suffix cutoff invalidates the selected position and its causal future:

~~~text
(PCA ID, fromPosition)
~~~

A Verifier or PIC-X MUST reject a matching state when the authenticated revocation state contains an authorized suffix cutoff and:

~~~text
state.PCA_ID == target.PCA_ID
AND state.position >= target.fromPosition
~~~

The exact wire representation and any concrete revocation-profile naming for this operation are profile-defined. This document defines only
the semantic causal cutoff.

## Whole PCA

Whole-PCA revocation invalidates all continuity states rooted in the selected PCA ID. It is equivalent to a suffix revocation from the root
position:

~~~text
(PCA ID, rootPosition)
~~~

## Optional Selectors

A revocation profile MAY define additional authenticated selectors, including:

- grant binding;
- origin issuer;
- key or verification method;
- executor or delegate;
- attestation;
- policy;
- resource, tenant, or security-domain selector;
- another explicitly profile-defined authenticated selector.

These selectors are administrative or evidentiary criteria. They do not replace `(PCA ID, position)` when causal descendants must be
invalidated.

Possession or knowledge of a selector value MUST NOT authorize revocation. For example, knowledge of a grant identifier, PCA ID, key
identifier, issuer identifier, or position is not a revocation credential.

## Historical Selector Resolution

When a selector refers to historical evidence that is not present in every descendant continuity state, a profile that wants causal
retroactive effect MUST resolve that selector to an authenticated affected position:

~~~text
KEY / DELEGATE / ATTESTATION / ISSUER / other selector
        |
        v
authenticated affected position
        |
        v
(PCA ID, fromPosition)
~~~

The resolver MUST authenticate the selector occurrence, the PCA ID, the position, the integrity of the evidence used for resolution, and the
authority to materialize the resulting cutoff.

A bare assertion that a selector occurred at a position MUST NOT be sufficient. A valid position witness MAY be an authenticated settled
continuity artifact, an authenticated transition extract, an authenticated historical-index record, a checkpoint, a Merkle inclusion proof, a
succinct proof, or another profile-defined authenticated witness.

For a compromised-key revocation, a position witness authenticated only by the compromised key MUST NOT be sufficient evidence that the
position was accepted into the continuity. The revocation authority MUST require corroboration independent of the revoked key, such as an
authenticated index, a trusted validator record, a checkpoint, or another profile-defined proof.

## Dynamic Restrictions

A profile MAY define dynamic authority or execution-contract restrictions. Such restrictions are evaluated as external authenticated state
against the presented PCA ID, position, and optional selectors.

Dynamic restrictions MUST be monotonic: they may only make effective authority or execution conformance equally or more restrictive. They
MUST NOT rewrite the signed root PIC PCA JWT, mutate historical transitions, alter predecessor uniqueness, replace Proof of Relationship, or
expand authority.

The profile MUST define restriction scope, composition, conflict resolution, freshness, rollback protection, and unavailable-state behavior.

# Revocation Authorization

This section is normative.

Every accepted revocation or dynamic restriction MUST be authenticated by an authorized revocation authority. A matching target coordinate is
necessary for targeting but never sufficient for authorization.

A profile MUST define how each revocation authority is proven. Possible authorities include:

| Revoker | Scope | Required proof |
| --- | --- | --- |
| Root authority or PCA issuer | the PCA ID it is authorized to administer | profile-defined issuer authorization and revocation proof |
| Grant authority | continuities bound to that grant | authenticated grant binding and grant-authority proof |
| Origin issuer | continuities whose profile authenticates that issuer binding | issuer revocation authority proof |
| Executor at position `k` | its own future, as permitted by policy | authenticated position proof and policy authority |
| Evidence authority | uses of a key, delegate, attestation, or issuer selector | selector proof plus authority to materialize a causal cutoff |
| Policy authority | dynamic policy or restriction state | authenticated policy authority proof |

An executor at position `k` MUST NOT revoke a position upstream of itself unless it separately holds root, grant, issuer, or other
profile-defined revocation authority. A policy MAY allow an executor to revoke its own future from `k+1`, or from `k` when the current
position itself must be invalidated.

Grant revocation MUST be accepted only when the Verifier can establish that the revocation issuer is authorized for the grant. Possession or
knowledge of a grant identifier alone MUST NOT authorize revocation.

# Revocation State

This section is normative.

Revocation state is external, authenticated, and append-only. A profile MUST specify:

- how revocation state is authenticated;
- how freshness is determined;
- how rollback is detected;
- who defines the minimum acceptable revocation-state version;
- how that minimum is authenticated;
- what happens when required revocation state is unavailable.

Unavailable-state behavior MUST be explicit. Examples include fail closed, accept only a fresh stapled proof, accept within a bounded offline
window, or another profile-defined behavior.

Security revocations and restrictions applied to a PCA ID, position, grant, selector, or policy scope MUST be append-only and MUST become
only equally or more restrictive over time:

~~~text
effectiveRevocationState(t + 1) is at least as restrictive as effectiveRevocationState(t)
~~~

A relaxation MUST require a new grant, a new trusted root authority context, a new PCA ID, or another profile-defined re-origination
mechanism. It MUST NOT be implemented by deleting an existing revocation, replaying an older revocation view, replacing active state with
less restrictive state, or silently un-revoking an existing cutoff.

# Future Branch-Capable Profiles

This section is non-normative.

PIC can support branch-capable revocation in a future or separate profile, but branch-domain machinery is not part of the current
centralized Profile 0.2 base coordinate. This document therefore does not define branch-domain fields, branch-suffix strategies,
branch-creation evidence, parent branch ancestry, or branch-domain escape rules for Profile 0.2.

A future branch-capable profile would need to define bounded authenticated branch coordinates, branch-creation authorization, anti-escape
validation order, branch-selective cutoff authorization, historical resolution rules, and privacy properties. Those details are intentionally
out of scope for this revision.

# Position Capacity

This appendix is non-normative. If a profile represents continuity position with a finite integer domain, exhausting that domain requires
completing that many causal advancements on one path:

~~~text
secondsToOverflow = (2^64 - 1 - currentPosition) / R
    where R = maximum accepted advancements per second on one continuity path
~~~

Each path is causally sequential; fan-out permits several paths but does not parallelize one path. A concrete `R` depends on hardware,
network, and application logic and is not fixed here. Under any realistic per-path rate a `uint64` position is practically inexhaustible,
though a profile that wants the mathematically unbounded case MAY use an unbounded integer encoding. The normative point is only that
overflow and wraparound are rejected when the selected representation can overflow.

# Contributors {#contributors}

The editors and contributors of this document are listed in the **document header** above. Listing is governed by Appendix B.7 of the
[PIC Legal Appendices](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-legal.md).

# Acknowledgement

The **Provenance Identity Continuity (PIC) Model** — the theoretical framework this specification expresses in normative form — was created
by **Nicola Gallo**. It first appeared on Zenodo on 1 December 2025 and is developed in full in the Proof-of-Continuity paper:

- Gallo, N. (2025). *PIC Model — Provenance Identity Continuity for Distributed Execution Systems*. Zenodo.
  [zenodo.org/records/17777421](https://zenodo.org/records/17777421) (DOI: 10.5281/zenodo.17777421).
- Gallo, N. (2026). *Proof-of-Continuity: A Temporal Model for Authority Propagation in Distributed Systems and AI Agents*. Zenodo.
  [zenodo.org/records/21285112](https://zenodo.org/records/21285112) (DOI: 10.5281/zenodo.21285112).

Authorship of the PIC Model remains with Nicola Gallo; the PIC specifications are published and maintained by **Nitro Agility S.r.l.** as
Specification Steward. Any work that references, implements, or claims conformance with PIC must preserve this attribution, distinguishing the
**PIC Model** (author: Nicola Gallo) from the **PIC Specifications** (steward: Nitro Agility S.r.l.), as required by the
**[PIC Legal Appendices](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-legal.md)** (Appendix B, Attribution; Appendix D,
Acknowledgements), which are incorporated into this specification by reference.

# Legal Notices

The appendices governing:

- **A.** Use of Automated Language Assistance,
- **B.** Authorship, Stewardship, Attribution, and Derivative Works,
- **C.** Disclaimer and Limitation of Liability,
- **D.** Acknowledgements,

are maintained in a single canonical document, the
**[PIC Legal Appendices](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-legal.md)** (`draft/0.2/pic-legal.md`), and are
**incorporated into this specification by reference** as if fully set forth herein.

In case of conflict between this document and the PIC Legal Appendices, the PIC Legal Appendices prevail for legal, governance, licensing,
and attribution matters.

This specification is subordinate to the
[PIC Specification](https://github.com/pic-protocol/pic-spec/blob/main/draft/0.2/pic-spec.md), which defines the normative semantics of the
PIC Model and is the entry point of the specification set. This document does not introduce new conceptual authority, invariants, or
authorship claims beyond those defined in the PIC Legal Appendices.

# References {#references}

- [1] Gallo, N. (2026). *Proof-of-Continuity: A Temporal Model for Authority Propagation in Distributed Systems and AI Agents*. arXiv:2607.08906 [cs.CR]. [arxiv.org/abs/2607.08906](https://arxiv.org/abs/2607.08906)
- [2] Bradner, S. (1997). *Key words for use in RFCs to Indicate Requirement Levels*. BCP 14, RFC 2119. [rfc-editor.org/rfc/rfc2119](https://www.rfc-editor.org/rfc/rfc2119)
- [3] Leiba, B. (2017). *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. BCP 14, RFC 8174. [rfc-editor.org/rfc/rfc8174](https://www.rfc-editor.org/rfc/rfc8174)
