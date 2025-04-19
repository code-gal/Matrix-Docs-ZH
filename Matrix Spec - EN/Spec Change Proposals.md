 > [Matrix Specification](https://spec.matrix.org/unstable/) |  [Spec Change Proposals](https://spec.matrix.org/unstable/proposals/)

# Spec Change Proposals

If you are interested in submitting a change to the Matrix Specification, please take note of the following guidelines.

Most changes to the Specification require a formal proposal. Bug fixes, typos, and clarifications to existing behaviour do not need proposals - see the [contributing guide](https://github.com/matrix-org/matrix-spec/blob/main/CONTRIBUTING.rst) for more information on what does and does not need a proposal.

The proposal process involves some technical writing, having it reviewed by everyone, having the proposal being accepted, then actually having your ideas implemented as committed changes to the [Specification repository](https://github.com/matrix-org/matrix-spec).

Meet the [members of the Core Team](https://matrix.org/foundation), a group of individuals tasked with ensuring the spec process is as smooth and painless as possible. Members of the Spec Core Team will do their best to participate in discussion, summarise when things become long-winded, and generally try to act towards the benefit of everyone. As a majority, team members have the ability to change the state of a proposal, and individually have the final say in proposal discussion.

## Guiding Principles[](https://spec.matrix.org/proposals/#guiding-principles)

Proposals **must** act to the greater benefit of the entire Matrix ecosystem, rather than benefiting or privileging any single player or subset of players -and must not contain any patent encumbered intellectual property. Members of the Core Team pledge to act as a neutral custodian for Matrix on behalf of the whole ecosystem.

For clarity: the Matrix ecosystem is anyone who uses the Matrix protocol. That includes client users, server admins, client developers, bot developers, bridge and application service developers, users and admins who are indirectly using Matrix via 3rd party networks which happen to be bridged, server developers, room moderators and admins, companies/projects building products or services on Matrix, spec contributors, translators, and those who created it in the first place.

“Greater benefit” could include maximising:

- the number of end-users reachable on the open Matrix network
- the number of regular users on the Matrix network (e.g. 30-day retained federated users)
- the number of online servers in the open federation
- the number of developers building on Matrix
- the number of independent implementations which use Matrix
- the number of bridged end-users reachable on the open Matrix network
- the signal-to-noise ratio of the content on the open Matrix network (i.e. minimising spam)
- the ability for users to discover content on their terms (empowering them to select what to see and what not to see)
- the quality and utility of the Matrix spec (as defined by ease and ability with which a developer can implement spec-compliant clients, servers, bots, bridges, and other integrations without needing to refer to any other external material)

In addition, proposal authors are expected to uphold the following values in their proposed changes to the Matrix protocol:

- Supporting the whole long-term ecosystem rather than individual stakeholder gain
- Openness rather than proprietary lock-in
- Interoperability rather than fragmentation
- Cross-platform rather than platform-specific
- Collaboration rather than competition
- Accessibility rather than elitism
- Transparency rather than stealth
- Empathy rather than contrariness
- Pragmatism rather than perfection
- Proof rather than conjecture

Please [see MSC1779](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/1779-open-governance.md) for full details of the project’s Guiding Principles.

## Technical notes[](https://spec.matrix.org/proposals/#technical-notes)

Proposals **must** develop Matrix as a layered protocol: with new features building on layers of shared abstractions rather than introducing tight vertical coupling within the stack. This ensures that new features can evolve rapidly by building on existing layers and swapping out old features without impacting the rest of the stack or requiring substantial upgrades to the whole ecosystem. This is critical for Matrix to rapidly evolve and compete effectively with centralised systems, despite being a federated protocol.

For instance, new features should be implemented using the highest layer abstractions possible (e.g. new event types, which layer on top of the existing room semantics, and so don’t even require any API changes). Failing that, the next recourse would be backwards-compatible changes to the next layer down (e.g. room APIs); failing that, considering changes to the format of events or the DAG; etc. It would be a very unusual feature which doesn’t build on the existing infrastructure provided by the spec and instead created new primitives or low level APIs.

Backwards compatibility is very important for Matrix, but not at the expense of hindering the protocol’s evolution. Backwards incompatible changes to endpoints are allowed when no other alternative exists, and must be versioned under a new major release of the API. Backwards incompatible changes to the room algorithm are also allowed when no other alternative exists, and must be versioned under a new version of the room algorithm.

There is sometimes a dilemma over where to include higher level features: for instance, should video conferencing be formalised in the spec, or should it be implemented via widgets? Should reputation systems be specified? Should search engine behaviour be specified?

There is no universal answer to this, but the following guidelines should be applied:

1. If the feature would benefit the whole Matrix ecosystem and is aligned with the guiding principles above, then it should be supported by the spec.
2. If the spec already makes the feature possible without changing any of the implementations and spec, then it may not need to be added to the spec.
3. However, if the best user experience for a feature does require custom implementation behaviour then the behaviour should be defined in the spec such that all implementations may implement it.
4. However, the spec must never add dependencies on unspecified/nonstandardised 3rd party behaviour.

As a worked example:

1. Video conferencing is clearly a feature which would benefit the whole ecosystem, and so the spec should find a way to make it happen.
2. Video conferencing can be achieved by widgets without requiring any compulsory changes to clients nor servers to work, and so could be omitted from the spec.
3. A better experience could be achieved by embedding Jitsi natively into clients rather than using a widget…
4. …except that would add a dependency on unspecified/nonstandardised 3rd party behaviour, so must not be added to the spec.

Therefore, our two options in the specific case of video conferencing are either to spec SFU conferencing semantics for WebRTC (or refer to an existing spec for doing so), or to keep it as a widget-based approach (optionally with widget extensions specific for more deeply integrating video conferencing use cases).

As an alternative example: it’s very unlikely that “how to visualise Magnetic Resonance Imaging data over Matrix” would ever be added to the Matrix spec (other than perhaps a custom event type in a wider standardised Matrix event registry) given that the spec’s existing primitives of file transfer and extensible events (MSC1767) give excellent tools for transferring and visualising arbitrary rich data.

Supporting public search engines are likely to not require custom spec features (other than possibly better bulk access APIs), given they can be implemented as clients using the existing CS API. An exception could be API features required by decentralised search infrastructure (avoiding centralisation of power by a centralised search engine).

Features such as reactions, threaded messages, editable messages, spam/abuse/content filtering (and reputation systems), are all features which would clearly benefit the whole Matrix ecosystem, and cannot be implemented in an interoperable way using the current spec; so they necessitate a spec change.

## Process[](https://spec.matrix.org/proposals/#process)

The process for submitting a Matrix Spec Change (MSC) Proposal in detail is as follows:

- Create a first draft of your proposal using [GitHub-flavored Markdown](https://help.github.com/articles/basic-writing-and-formatting-syntax/)
    - In the document, clearly state the problem being solved, and the possible solutions being proposed for solving it and their respective trade-offs.
    - Proposal documents are intended to be as lightweight and flexible as the author desires; there is no formal template; the intention is to iterate as quickly as possible to get to a good design.
    - However, a [template with suggested headers](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/0000-proposal-template.md) is available to get you started if necessary.
    - Take care in creating your proposal. Specify your intended changes, and give reasoning to back them up. Changes without justification will likely be poorly received by the community.
- Fork and make a PR to the [matrix-spec-proposals](https://github.com/matrix-org/matrix-spec-proposals) repository. The ID of your PR will become the MSC ID for the lifetime of your proposal.
    - The proposal must live in the `proposals/` directory with a filename that follows the format `1234-my-new-proposal.md` where `1234` is the MSC ID.
    - Your PR description must include a link to the rendered Markdown document and a summary of the proposal.
    - It is often very helpful to link any related MSCs or [matrix-spec issues](https://github.com/matrix-org/matrix-spec/issues) to give context for the proposal.
    - Additionally, please be sure to sign off your proposal PR as per the guidelines listed on [CONTRIBUTING.rst](https://github.com/matrix-org/matrix-spec/blob/main/CONTRIBUTING.rst).
- Gather feedback as widely as possible.
    - The aim is to get maximum consensus towards an optimal solution. Sometimes trade-offs are required to meet this goal. Decisions should be made to the benefit of all major use cases.
    - A good place to ask for feedback on a specific proposal is [#matrix-spec:matrix.org](https://matrix.to/#/#matrix-spec:matrix.org). If preferred, an alternative room can be created and advertised in #matrix-spec:matrix.org. Please also link to the room in your PR description.
    - For additional discussion areas, know that #matrix-dev:matrix.org is for developers using existing Matrix APIs, #matrix:matrix.org is for users trying to run Matrix apps (clients & servers) and #matrix-architecture:matrix.org is for cross-cutting discussion of Matrix’s architectural design.
    - The point of the spec proposal process is to be collaborative rather than competitive, and to try to solve the problem in question with the optimal set of trade-offs. The author should neutrally gather the various viewpoints and get consensus, but this can sometimes be time-consuming (or the author may be biased), in which case an impartial ‘shepherd’ can be assigned to help guide the proposal through this process instead. A shepherd is typically a neutral party from the Spec Core Team or an experienced member of the community. There is no formal process for assignment. Simply ask for a shepherd to help get your proposal through and one will be assigned based on availability. Having a shepherd is not a requirement for proposal acceptance.
- Members of the Spec Core Team and community will review and discuss the PR in the comments and in relevant rooms on Matrix. Discussion outside of GitHub should be summarised in a comment on the PR.
- When a member of the Spec Core Team believes that no new discussion points are being made, and the proposal has suitable evidence of working (see [implementing a proposal](https://spec.matrix.org/proposals/#implementing-a-proposal) below), they will propose a motion for a final comment period (FCP), along with a _disposition_ of either merge, close or postpone. This FCP is provided to allow a short period of time for any invested party to provide a final objection before a major decision is made. If sufficient reasoning is given, an FCP can be cancelled. It is often preceded by a comment summarising the current state of the discussion, along with reasoning for its occurrence.
- A concern can be raised by a Spec Core Team member at any time, which will block an FCP from beginning. An FCP will only begin when 75% of the members of the Spec Core Team agree on its outcome, and all existing concerns have been resolved.
- The FCP will then begin and last for 5 days, giving anyone else some time to speak up before it concludes. If sufficient reasoning against the disposition is provided, a member of the Spec Core Team can raise a concern and block FCP from completing. This will not reset or pause the 5 day FCP timer, but FCP will not conclude until all concerns have been resolved. If sufficient change in the MSC is required to resolve those concerns, FCP might be cancelled and reproposed. Once FCP has concluded, the disposition of the FCP will be carried out.
- Once the proposal has been accepted and merged, it is time to submit the actual change to the Specification that your proposal reasoned about. This is known as a spec PR. However in order for the spec PR to be accepted, an implementation **must** be shown to prove that it works well in practice. A link to the implementation should be included in the PR description. In addition, any significant unforeseen changes to the original idea found during this process will warrant another MSC. Any minor, non-fundamental changes are allowed but **must** be documented in the original proposal document. This ensures that someone reading a proposal in the future doesn’t assume old information that wasn’t merged into the spec.
    - Similar to the proposal PR, please sign off the spec PR as per the guidelines on [CONTRIBUTING.rst](https://github.com/matrix-org/matrix-spec/blob/main/CONTRIBUTING.rst).
- Your PR will then be reviewed and hopefully merged on the grounds it is implemented sufficiently. If so, then give yourself a pat on the back knowing you’ve contributed to the Matrix protocol for the benefit of users and developers alike :)

The process for handling proposals is shown visually in the following diagram. Note that the lifetime of a proposal is tracked through the corresponding labels for each stage on the [matrix-spec-proposals](https://github.com/matrix-org/matrix-spec-proposals) pull request trackers.

```
                           +                          +
         Proposals         |          Spec PRs        |  Additional States
         +-------+         |          +------+        |  +---------------+
                           |                          |
 +----------------------+  |         +---------+      |    +-----------+
 |                      |  |         |         |      |    |           |
 |      Proposal        |  |  +------= Spec PR |      |    | Postponed |
 | Drafting and Initial |  |  |      | Missing |      |    |           |
 |  Feedback Gathering  |  |  |      |         |      |    +-----------+
 |                      |  |  |      +----+----+      |
 +----------+-----------+  |  |           |           |    +----------+
            |              |  |           v           |    |          |
            v              |  |  +-----------------+  |    |  Closed  |
  +-------------------+    |  |  |                 |  |    |          |
  |                   |    |  |  | Spec PR Created |  |    +----------+
  |    Proposal PR    |    |  |  |  and In Review  |  |
  |     In Review     |    |  |  |                 |  |
  |                   |    |  |  +--------+--------+  |
  +---------+---------+    |  |           |           |
            |              |  |           v           |
            v              |  |     +-----------+     |
 +----------------------+  |  |     |           |     |
 |                      |  |  |     |  Spec PR  |     |
 |    Proposed Final    |  |  |     |  Merged!  |     |
 |    Comment Period    |  |  |     |           |     |
 |                      |  |  |     +-----------+     |
 +----------+-----------+  |  |                       |
            |              |  |                       |
            v              |  |                       |
 +----------------------+  |  |                       |
 |                      |  |  |                       |
 | Final Comment Period |  |  |                       |
 |                      |  |  |                       |
 +----------+-----------+  |  |                       |
            |              |  |                       |
            v              |  |                       |
 +----------------------+  |  |                       |
 |                      |  |  |                       |
 | Final Comment Period |  |  |                       |
 |       Complete       |  |  |                       |
 |                      |  |  |                       |
 +----------+-----------+  |  |                       |
            |              |  |                       |
            +-----------------+                       |
                           |                          |
                           +                          +
```

## Lifetime States[](https://spec.matrix.org/proposals/#lifetime-states)

**Note:** All labels are to be placed on the proposal PR.

|Name|GitHub Label|Description|
|---|---|---|
|Proposal Drafting and Feedback|[Draft pull request](https://github.com/matrix-org/matrix-spec-proposals/issues?q=is:open+draft:true)|A proposal document which is still work-in-progress but is being shared to incorporate feedback. Please prefix your proposal’s title with `[WIP]` to make it easier for reviewers to skim their notifications list.|
|Proposal In Review|[No label](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+-label%3Aabandoned+-label%3Afinal-comment-period+-label%3Afinished-final-comment-period+-label%3Amerged+-label%3Aobsolete+-label%3Aproposal-postponed+-label%3Aproposed-final-comment-period+-label%3Aproposal-in-review+-label%3Aspec-pr-in-review+-label%3Aspec-pr-missing)|A proposal document which is now ready and waiting for review by the Spec Core Team and community|
|Proposed Final Comment Period|[proposed-final-comment-period](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aproposed-final-comment-period+)|Currently awaiting signoff of a 75% majority of team members in order to enter the final comment period|
|Final Comment Period|[final-comment-period](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Afinal-comment-period+)|A proposal document which has reached final comment period either for merge, closure or postponement|
|Final Comment Period Complete|[finished-final-comment-period](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Afinished-final-comment-period+)|The final comment period has been completed. Waiting for a demonstration implementation|
|Spec PR Missing|[spec-pr-missing](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aspec-pr-missing)|The proposal has been agreed, and proven with a demonstration implementation. Waiting for a PR against the Spec|
|Spec PR In Review|[spec-pr-in-review](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aspec-pr-in-review+)|The spec PR has been written, and is currently under review|
|Spec PR Merged|[merged](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Amerged)|A proposal with a sufficient working implementation and whose Spec PR has been merged!|
|Postponed|[proposal-postponed](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aproposal-postponed+)|A proposal that is temporarily blocked or a feature that may not be useful currently but perhaps sometime in the future|
|Abandoned|[abandoned](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aabandoned)|A proposal where the author/shepherd is not responsive|
|Obsolete|[obsolete](https://github.com/matrix-org/matrix-spec-proposals/issues?q=draft:false+label%3Aproposal+label%3Aobsolete+)|A proposal which has been made obsolete by another proposal or decision elsewhere.|

## Categories[](https://spec.matrix.org/proposals/#categories)

We use category labels on MSCs to place them into a track of work. The Spec Core Team decides which of the tracks they are focusing on for the next while and generally makes an effort to pull MSCs out of that category when possible.

The current categories are:

|Name|GitHub Label|Description|
|---|---|---|
|Core|kind:core|Important for the protocol’s success.|
|Feature|kind:feature|Nice to have additions to the spec.|
|Maintenance|kind:maintenance|Fixes or clarifies existing spec.|

Some examples of core MSCs would be aggregations, cross-signing, and groups/communities. These are the sorts of things that if not implemented could cause the protocol to fail or become second-class. Features would be areas like enhanced media APIs, new transports, and bookmarks in comparison. Finally, maintenance MSCs would include improving error codes, clarifying what is required of an API, and adding properties to an API which makes it easier to use.

The Spec Core Team assigns a category to each MSC based on the descriptions above. This can mean that new MSCs get categorized into an area the team isn’t focused on, though that can always change as priorities evolve. We still encourage that MSCs be opened, even if not the focus for the time being, as they can still make progress and even be merged without the Spec Core Team focusing on them specifically.

## Implementing a proposal[](https://spec.matrix.org/proposals/#implementing-a-proposal)

As part of the proposal process the Spec Core Team will require evidence of the MSC working in order for it to move into FCP. This can usually be a branch/pull request to whichever implementation of choice that proves the MSC works in practice, though in some cases the MSC itself will be small enough to be considered proven. Implementations do not need to be merged or released, but must be of sufficient quality to show that the MSC works. Where it’s unclear if an MSC will require an implementation proof, ask in [#matrix-spec:matrix.org](https://matrix.to/#/#matrix-spec:matrix.org). Proposals may require both server-side and client-side implementations.

Proposals that have not yet been implemented will have the `needs-implementation` label. After an implementation has been made, add a comment in the GitHub issue indicating so. After an implementation has been made, we will check it to verify that it implements the MSC. Proposals that have implementations that have not yet been checked will have the `implementation-needs-checking` label.

### Early release of an MSC/idea[](https://spec.matrix.org/proposals/#early-release-of-an-mscidea)

To help facilitate early releases of software dependent on a spec release, implementations are required to use the following process to ensure that the official Matrix namespace is not cluttered with development or testing data.

**Note:** Unreleased implementations (including proofs-of-concept demonstrating that a particular MSC works) do not have to follow this process.

1. Have an idea for a feature.
2. Implement the feature using unstable endpoints, vendor prefixes, and unstable feature flags as appropriate.
    - When using unstable endpoints, they MUST include a vendor prefix. For example: `/_matrix/client/unstable/com.example/login`. Vendor prefixes throughout Matrix always use the Java package naming convention. The MSC for the feature should identify which preferred vendor prefix is to be used by early adopters.
    - Note that unstable namespaces do not automatically inherit endpoints from stable namespaces: for example, the fact that `/_matrix/client/r0/sync` exists does not imply that `/_matrix/client/unstable/com.example/sync` exists.
    - If the client needs to be sure the server supports the feature, an unstable feature flag that MUST be vendor prefixed is to be used. This kind of flag shows up in the `unstable_features` section of `/versions` as, for example, `com.example.new_login`. The MSC for the feature should identify which preferred feature flag is to be used by early adopters.
    - When using this approach correctly, the implementation can ship/release the feature at any time, so long as the implementation is able to accept the technical debt that results from needing to provide adequate backwards and forwards compatibility. The implementation MUST support the flag (and server-side implementation) disappearing and be generally safe for users. Note that implementations early in the MSC review process may also be required to provide backwards compatibility with earlier editions of the proposal.
    - If the implementation cannot support the technical debt (or if it’s impossible to provide forwards/backwards compatibility - e.g. a user authentication change which can’t be safely rolled back), the implementation should not attempt to implement the feature and should instead wait for a spec release.
    - If at any point after early release, the idea changes in a backwards-incompatible way, the feature flag should also change so that implementations can adapt as needed.
3. In parallel, or ahead of implementation, open an MSC and solicit review per above.
4. Before FCP can be called, the Spec Core Team will require evidence of the MSC working as proposed. A typical example of this is an implementation of the MSC, though the implementation does not need to be shipped anywhere and can therefore avoid the forwards/backwards compatibility concerns mentioned here.
5. The FCP process is completed, and assuming nothing is flagged the MSC lands.
6. Implementations can now switch to using stable prefixes (for example, for an endpoint, moving from `/unstable/org.matrix.mscxxxx/frobnicate` to `/v1/frobnicate`), assuming that the change is backwards compatible with older implementations. In the rare occasion where backwards compatibility is not possible without a new spec release, implementations should continue to use unstable prefixes.
7. A spec PR is written to incorporate the changes into Matrix.
8. A spec release happens.
9. A transition period of about 2 months starts immediately after the spec release, before implementations start to encourage other implementations to switch to stable endpoints. For example, a server implementation should start asking client implementations to support the stable endpoints 2 months after the spec release, if they haven’t already. The same applies in the reverse: if clients cannot switch to stable prefixes because server implementations haven’t started supporting the new spec release, some noise should be raised in the general direction of the implementation.

> [!info] info:
> MSCs MUST still describe what the stable endpoints/feature looks like with a note towards the bottom for what the unstable feature flag/prefixes are. For example, an MSC would propose `/_matrix/client/r0/new/endpoint`, not `/_matrix/client/unstable/ com.example/new/endpoint`.

In summary:

- Implementations MUST NOT use stable endpoints before the MSC has completed FCP. Once that has occurred, implementations are allowed to use stable endpoints, but are not required to.
- Implementations are able to ship features that are exposed to users by default before an MSC has been merged to the spec, provided they follow the process above.
- Implementations SHOULD be wary of the technical debt they are incurring by moving faster than the spec.
- The vendor prefix is chosen by the developer of the feature, using the Java package naming convention. The foundation’s preferred vendor prefix is `org.matrix`.
- The vendor prefixes, unstable feature flags, and unstable endpoints should be included in the MSC, though the MSC MUST be written in a way that proposes new stable endpoints. Typically this is solved by a small table at the bottom mapping the various values from stable to unstable.

## Proposal Tracking[](https://spec.matrix.org/proposals/#proposal-tracking)

This is a living document generated from the list of proposals on the issue and pull request trackers of the [matrix-spec-proposals](https://github.com/matrix-org/matrix-spec-proposals) repo.

We use labels and some metadata in MSC PR descriptions to generate this page. Labels are assigned by the Spec Core Team whilst triaging the proposals based on those which exist in the [matrix-spec-proposals](https://github.com/matrix-org/matrix-spec-proposals) repo already.

It is worth mentioning that a previous version of the MSC process used a mixture of GitHub issues and PRs, leading to some MSC numbers deriving from GitHub issue IDs instead. A useful feature of GitHub is that it does automatically resolve to an issue, if an issue ID is placed in a pull URL. This means that [https://github.com/matrix-org/matrix-spec-proposals/pull/$MSCID](https://github.com/matrix-org/matrix-spec-proposals/pull/$MSCID) will correctly resolve to the desired MSC, whether it started as an issue or a PR.

Other metadata:

- The MSC number is taken from the GitHub Pull Request ID. This is carried for the lifetime of the proposal. These IDs do not necessary represent a chronological order.
- The GitHub PR title will act as the MSC’s title.
- Please link to the spec PR (if any) by adding a “PRs: #1234” line in the issue description.
- The creation date is taken from the GitHub PR, but can be overridden by adding a “Date: yyyy-mm-dd” line in the PR description.
- Updated Date is taken from GitHub.
- Author is the creator of the MSC PR, but can be overridden by adding a “Author: @username” line in the body of the issue description. Please make sure @username is a GitHub user (include the @!)
- A shepherd can be assigned by adding a “Shepherd: @username” line in the issue description. Again, make sure this is a real GitHub user.

### Work In Progress[](https://spec.matrix.org/proposals/#work-in-progress)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4192](https://github.com/matrix-org/matrix-spec-proposals/pull/4192)|Comparison of proposals for ignoring invites|2024-09-13|2024-09-13||[@Johennes](https://github.com/Johennes)||
|[4191](https://github.com/matrix-org/matrix-spec-proposals/pull/4191)|MSC4191: Account management deep-linking|2024-09-12|2024-09-12||[@sandhose](https://github.com/sandhose)||
|[4179](https://github.com/matrix-org/matrix-spec-proposals/pull/4179)|MSC4179: Moderation event hiding|2024-08-19|2024-09-14||[@tranquillity-codes](https://github.com/tranquillity-codes)||
|[4169](https://github.com/matrix-org/matrix-spec-proposals/pull/4169)|MSC4169: Backwards-compatible redaction sending using `/send`|2024-07-07|2024-07-07||[@tulir](https://github.com/tulir)||
|[4157](https://github.com/matrix-org/matrix-spec-proposals/pull/4157)|MSC4157: Futures (widget api)|2024-06-19|2024-08-01||[@toger5](https://github.com/toger5)||
|[4152](https://github.com/matrix-org/matrix-spec-proposals/pull/4152)|[WIP] MSC4152: Room labeling and filtering|2024-06-05|2024-06-10||[@turt2live](https://github.com/turt2live)||
|[4145](https://github.com/matrix-org/matrix-spec-proposals/pull/4145)|MSC4145: Simple verified accounts|2024-05-14|2024-05-30||[@turt2live](https://github.com/turt2live)||
|[4143](https://github.com/matrix-org/matrix-spec-proposals/pull/4143)|MSC4143: MatrixRTC|2024-05-10|2024-09-11||[@toger5](https://github.com/toger5)||
|[4139](https://github.com/matrix-org/matrix-spec-proposals/pull/4139)|MSC4139: Bot buttons & conversations|2024-05-06|2024-05-06||[@turt2live](https://github.com/turt2live)||
|[4109](https://github.com/matrix-org/matrix-spec-proposals/pull/4109)|MSC4109: Appservices & soft-failed events|2024-02-23|2024-02-23||[@turt2live](https://github.com/turt2live)||
|[4107](https://github.com/matrix-org/matrix-spec-proposals/pull/4107)|MSC4107: Feature-focused versioning|2024-02-19|2024-02-23||[@turt2live](https://github.com/turt2live)||
|[4106](https://github.com/matrix-org/matrix-spec-proposals/pull/4106)|MSC4106: Join as Muted|2024-02-19|2024-02-27||[@FSG-Cat](https://github.com/FSG-Cat)||
|[4104](https://github.com/matrix-org/matrix-spec-proposals/pull/4104)|MSC4104: Auth Lock: Soft-failure-be-gone!|2024-02-19|2024-02-19||[@Gnuxie](https://github.com/Gnuxie)||
|[4101](https://github.com/matrix-org/matrix-spec-proposals/pull/4101)|MSC4101: Hashes for unencrypted media|2024-02-15|2024-02-20||[@turt2live](https://github.com/turt2live)||
|[4100](https://github.com/matrix-org/matrix-spec-proposals/pull/4100)|MSC4100: Scoped signing keys|2024-02-14|2024-02-15||[@turt2live](https://github.com/turt2live)||
|[4099](https://github.com/matrix-org/matrix-spec-proposals/pull/4099)|MSC4099: Participation based authorization for servers in the Matrix DAG|2024-02-12|2024-02-15||[@Gnuxie](https://github.com/Gnuxie)||
|[4097](https://github.com/matrix-org/matrix-spec-proposals/pull/4097)|[WIP] MSC4097: Interactions between media redirection and authentication|2024-02-06|2024-07-29||[@turt2live](https://github.com/turt2live)||
|[4094](https://github.com/matrix-org/matrix-spec-proposals/pull/4094)|[WIP] MSC4094: Sync Server and Client Times with endpoint|2024-01-29|2024-08-05||[@toger5](https://github.com/toger5)||
|[4089](https://github.com/matrix-org/matrix-spec-proposals/pull/4089)|MSC4089: Delivery Receipts|2024-01-12|2024-05-30||[@turt2live](https://github.com/turt2live)||
|[4086](https://github.com/matrix-org/matrix-spec-proposals/pull/4086)|MSC4086: Event media reference counting|2023-12-23|2023-12-26||[@AndrewRyanChama](https://github.com/AndrewRyanChama)||
|[4080](https://github.com/matrix-org/matrix-spec-proposals/pull/4080)|MSC4080: Cryptographic Identities (Client-Owned Identities)|2023-11-15|2024-05-29||[@devonh](https://github.com/devonh)||
|[4060](https://github.com/matrix-org/matrix-spec-proposals/pull/4060)|MSC4060: Accept room rules before speaking|2023-10-14|2023-11-16||[@turt2live](https://github.com/turt2live)||
|[4059](https://github.com/matrix-org/matrix-spec-proposals/pull/4059)|[WIP] MSC4059: Mutable event content|2023-10-14|2024-05-30||[@turt2live](https://github.com/turt2live)||
|[4058](https://github.com/matrix-org/matrix-spec-proposals/pull/4058)|[WIP] MSC4058: Additive events|2023-10-14|2023-10-14||[@turt2live](https://github.com/turt2live)||
|[4057](https://github.com/matrix-org/matrix-spec-proposals/pull/4057)|MSC4057: Static Room Directory|2023-09-22|2023-09-23||[@networkException](https://github.com/networkException)||
|[4056](https://github.com/matrix-org/matrix-spec-proposals/pull/4056)|MSC4056: Role-Based Access Control (mk II)|2023-09-15|2024-08-27||[@turt2live](https://github.com/turt2live)||
|[4051](https://github.com/matrix-org/matrix-spec-proposals/pull/4051)|[WIP] MSC4051: Using the create event as the room ID|2023-09-03|2024-07-02||[@turt2live](https://github.com/turt2live)||
|[4049](https://github.com/matrix-org/matrix-spec-proposals/pull/4049)|MSC4049: Sending events as a server or room|2023-08-24|2024-05-30||[@turt2live](https://github.com/turt2live)||
|[4047](https://github.com/matrix-org/matrix-spec-proposals/pull/4047)|MSC4047: Send Keys|2023-08-23|2024-05-30||[@turt2live](https://github.com/turt2live)||
|[4046](https://github.com/matrix-org/matrix-spec-proposals/pull/4046)|MSC4046: Make & send PDU endpoints|2023-08-22|2023-08-23||[@turt2live](https://github.com/turt2live)||
|[4044](https://github.com/matrix-org/matrix-spec-proposals/pull/4044)|MSC4044: Enforcing user ID grammar in rooms|2023-08-12|2023-08-18||[@turt2live](https://github.com/turt2live)||
|[4038](https://github.com/matrix-org/matrix-spec-proposals/pull/4038)|MSC4038: Key backup for MLS|2023-07-19|2023-07-19||[@uhoreg](https://github.com/uhoreg)||
|[4031](https://github.com/matrix-org/matrix-spec-proposals/pull/4031)|MSC4031: Pre-generating invites and room invite codes|2023-06-18|2023-06-29||[@mdnwvn](https://github.com/mdnwvn)||
|[4029](https://github.com/matrix-org/matrix-spec-proposals/pull/4029)|MSC4029: [WIP] Fixing `X-Matrix` request authentication|2023-06-14|2024-06-10||[@turt2live](https://github.com/turt2live)||
|[4021](https://github.com/matrix-org/matrix-spec-proposals/pull/4021)|MSC4021: Archive client controls|2023-05-28|2023-07-05||[@jonaharagon](https://github.com/jonaharagon)||
|[4011](https://github.com/matrix-org/matrix-spec-proposals/pull/4011)|MSC4011: Thumbnail media negotiation|2023-05-04|2023-12-16||[@clokep](https://github.com/clokep), [@Half-Shot](https://github.com/Half-Shot)||
|[4002](https://github.com/matrix-org/matrix-spec-proposals/pull/4002)|MSC4002: Walkie talkie|2023-04-25|2023-04-28||[@tajo48](https://github.com/tajo48)||
|[3995](https://github.com/matrix-org/matrix-spec-proposals/pull/3995)|MSC3995: [WIP] Linearized Matrix|2023-04-10|2023-08-19||[@turt2live](https://github.com/turt2live)||
|[3994](https://github.com/matrix-org/matrix-spec-proposals/pull/3994)|MSC3994: Display why an event caused a notification|2023-04-06|2023-04-11||[@kerryarchibald](https://github.com/kerryarchibald)||
|[3964](https://github.com/matrix-org/matrix-spec-proposals/pull/3964)|MSC3964: Notifications for room tags|2023-02-03|2023-02-03||[@clokep](https://github.com/clokep)||
|[3956](https://github.com/matrix-org/matrix-spec-proposals/pull/3956)|[WIP] MSC3956: Extensible Events - Encrypted Events|2023-01-13|2023-04-18||[@turt2live](https://github.com/turt2live)||
|[3953](https://github.com/matrix-org/matrix-spec-proposals/pull/3953)|MSC3953: Server capability DAG|2023-01-07|2023-01-08||[@Gnuxie](https://github.com/Gnuxie)||
|[3948](https://github.com/matrix-org/matrix-spec-proposals/pull/3948)|MSC3948: Repository room for Thirdroom|2022-12-14|2023-06-28||[@ajbura](https://github.com/ajbura)||
|[3944](https://github.com/matrix-org/matrix-spec-proposals/pull/3944)|MSC3944: Dropping stale send-to-device messages|2022-12-06|2023-02-27||[@uhoreg](https://github.com/uhoreg)||
|[3935](https://github.com/matrix-org/matrix-spec-proposals/pull/3935)|MSC3935: Cute Events against social distancing|2022-11-16|2024-04-01||[@TheOneWithTheBraid](https://github.com/TheOneWithTheBraid)||
|[3909](https://github.com/matrix-org/matrix-spec-proposals/pull/3909)|MSC3909: Membership based mutes|2022-10-13|2024-07-12||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3902](https://github.com/matrix-org/matrix-spec-proposals/pull/3902)|MSC3902: Faster remote room joins over federation (overview)|2022-10-03|2024-06-29||[@richvdh](https://github.com/richvdh)||
|[3901](https://github.com/matrix-org/matrix-spec-proposals/pull/3901)|MSC3901: Deleting state|2022-09-28|2024-09-01||[@andybalaam](https://github.com/andybalaam)||
|[3898](https://github.com/matrix-org/matrix-spec-proposals/pull/3898)|[WIP] MSC3898: Native Matrix VoIP signalling for cascaded foci (SFUs, MCUs...)|2022-09-25|2023-01-03||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3895](https://github.com/matrix-org/matrix-spec-proposals/pull/3895)|MSC3895: Federation API Behaviour of Partial-State Resident Servers|2022-09-22|2022-09-30||[@reivilibre](https://github.com/reivilibre)||
|[3883](https://github.com/matrix-org/matrix-spec-proposals/pull/3883)|MSC3883: Fundamental state changes|2022-09-06|2022-09-09||[@timokoesters](https://github.com/timokoesters)||
|[3872](https://github.com/matrix-org/matrix-spec-proposals/pull/3872)|MSC3872: Order of rooms in Spaces|2022-08-20|2022-08-25||[@aMyTimed](https://github.com/aMyTimed)||
|[3868](https://github.com/matrix-org/matrix-spec-proposals/pull/3868)|[WIP] MSC3868: Room Contribution|2022-08-15|2022-08-18||[@jae1911](https://github.com/jae1911)||
|[3865](https://github.com/matrix-org/matrix-spec-proposals/pull/3865)|[WIP] MSC3865: User-given attributes for users|2022-08-09|2024-07-17||[@Zocker1999NET](https://github.com/Zocker1999NET)||
|[3864](https://github.com/matrix-org/matrix-spec-proposals/pull/3864)|[WIP] MSC3864: User-given attributes for rooms|2022-08-07|2024-03-31||[@Zocker1999NET](https://github.com/Zocker1999NET)||
|[3861](https://github.com/matrix-org/matrix-spec-proposals/pull/3861)|MSC3861: Next-generation auth for Matrix, based on OAuth 2.0/OIDC|2022-08-04|2024-09-13||[@hughns](https://github.com/hughns)||
|[3849](https://github.com/matrix-org/matrix-spec-proposals/pull/3849)|MSC3849: Observations and Reinforcement|2022-07-19|2024-05-21||[@Gnuxie](https://github.com/Gnuxie)||
|[3842](https://github.com/matrix-org/matrix-spec-proposals/pull/3842)|[WIP] MSC3842: Placeholder for power levels & extensible events|2022-07-02|2023-07-25||[@turt2live](https://github.com/turt2live)||
|[3839](https://github.com/matrix-org/matrix-spec-proposals/pull/3839)|MSC3839: primary-identity-as-key|2022-06-26|2022-12-15||[@zander](https://github.com/zander)||
|[3815](https://github.com/matrix-org/matrix-spec-proposals/pull/3815)|[WIP] MSC3815: 3D Worlds|2022-05-13|2023-06-28||[@robertlong](https://github.com/robertlong)||
|[3814](https://github.com/matrix-org/matrix-spec-proposals/pull/3814)|[WIP] MSC3814: Dehydrated devices with SSSS|2022-05-12|2024-05-22||[@uhoreg](https://github.com/uhoreg)||
|[3813](https://github.com/matrix-org/matrix-spec-proposals/pull/3813)|MSC3813: Obfuscated events|2022-05-10|2022-07-21||[@tusooa](https://github.com/tusooa)||
|[3775](https://github.com/matrix-org/matrix-spec-proposals/pull/3775)|MSC3775: Markup Locations for Audiovisual Media|2022-04-17|2022-09-16||[@gleachkr](https://github.com/gleachkr)||
|[3761](https://github.com/matrix-org/matrix-spec-proposals/pull/3761)|MSC3761: State event change control|2022-03-31|2022-04-05||[@turt2live](https://github.com/turt2live)||
|[3755](https://github.com/matrix-org/matrix-spec-proposals/pull/3755)|MSC3755: Member pronouns|2022-03-18|2023-06-02||[@Gnuxie](https://github.com/Gnuxie)||
|[3752](https://github.com/matrix-org/matrix-spec-proposals/pull/3752)|MSC3752: Markup locations for text|2022-03-09|2022-08-09||[@gleachkr](https://github.com/gleachkr)||
|[3726](https://github.com/matrix-org/matrix-spec-proposals/pull/3726)|[WIP] MSC3726: Safer Password-based Authentication with BS-SPEKE|2022-02-15|2022-03-27||[@cvwright](https://github.com/cvwright)||
|[3662](https://github.com/matrix-org/matrix-spec-proposals/pull/3662)|[WIP] MSC3662: Allow Widgets to share user mxids to the client|2022-01-20|2022-01-20||[@Half-Shot](https://github.com/Half-Shot)||
|[3647](https://github.com/matrix-org/matrix-spec-proposals/pull/3647)|[WIP] MSC3647: Bring Your Own Bridge - Decentralising Bridges|2022-01-14|2022-01-23||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3639](https://github.com/matrix-org/matrix-spec-proposals/pull/3639)|[WIP] MSC3639: Matrix for the social media use case|2022-01-12|2023-01-02||[@henri2h](https://github.com/henri2h)||
|[3592](https://github.com/matrix-org/matrix-spec-proposals/pull/3592)|MSC3592: Markup locations for PDF documents|2021-12-23|2022-05-18||[@gleachkr](https://github.com/gleachkr)||
|[3414](https://github.com/matrix-org/matrix-spec-proposals/pull/3414)|[WIP] MSC3414: Encrypted state events|2021-09-27|2023-06-05||[@turt2live](https://github.com/turt2live)||
|[3385](https://github.com/matrix-org/matrix-spec-proposals/pull/3385)|[WIP] MSC3385: Bulk small improvements to room upgrades|2021-09-07|2022-01-21||[@turt2live](https://github.com/turt2live)||
|[3361](https://github.com/matrix-org/matrix-spec-proposals/pull/3361)|[WIP] MSC3361: Opportunistic Direct Push|2021-08-25|2023-03-03||[@lukaslihotzki](https://github.com/lukaslihotzki)||
|[3359](https://github.com/matrix-org/matrix-spec-proposals/pull/3359)|[WIP] MSC3359: Delayed Push|2021-08-25|2023-07-14||[@jcgruenhage](https://github.com/jcgruenhage)||
|[3269](https://github.com/matrix-org/matrix-spec-proposals/pull/3269)|MSC3269: An error code for busy servers|2021-07-06|2021-08-30||[@Yoric](https://github.com/Yoric)||
|[3262](https://github.com/matrix-org/matrix-spec-proposals/pull/3262)|[WIP] MSC3262 aPAKE authentication|2021-07-02|2022-07-26||[@mvgorcum](https://github.com/mvgorcum)||
|[3219](https://github.com/matrix-org/matrix-spec-proposals/pull/3219)|[WIP] MSC3219: Space Flair|2021-05-27|2022-10-21||[@turt2live](https://github.com/turt2live)||
|[3215](https://github.com/matrix-org/matrix-spec-proposals/pull/3215)|Draft: MSC3215: Aristotle - Moderation in all things|2021-05-24|2024-05-20||[@Yoric](https://github.com/Yoric)||
|[3192](https://github.com/matrix-org/matrix-spec-proposals/pull/3192)|MSC3192: Batch state endpoint|2021-05-12|2024-05-20||[@clokep](https://github.com/clokep)||
|[3189](https://github.com/matrix-org/matrix-spec-proposals/pull/3189)|MSC3189: Per-room/per-space profiles|2021-05-11|2023-04-15||[@robintown](https://github.com/robintown)||
|[3184](https://github.com/matrix-org/matrix-spec-proposals/pull/3184)|MSC3184: Challenges Messages|2021-05-10|2024-05-18||[@BillCarsonFr](https://github.com/BillCarsonFr)||
|[3089](https://github.com/matrix-org/matrix-spec-proposals/pull/3089)|MSC3089: File tree structures|2021-04-02|2021-09-10||[@turt2live](https://github.com/turt2live)||
|[3088](https://github.com/matrix-org/matrix-spec-proposals/pull/3088)|MSC3088: Room subtyping|2021-04-01|2022-01-22||[@turt2live](https://github.com/turt2live)||
|[3062](https://github.com/matrix-org/matrix-spec-proposals/pull/3062)|MSC3062: Bot verification|2021-03-12|2024-02-23||[@uhoreg](https://github.com/uhoreg)||
|[3032](https://github.com/matrix-org/matrix-spec-proposals/pull/3032)|MSC3032: Thoughts on updating presence|2021-02-25|2021-08-30||[@dbkr](https://github.com/dbkr)||
|[3020](https://github.com/matrix-org/matrix-spec-proposals/pull/3020)|MSC3020: WIP proposal for better supporting private federations|2021-02-21|2024-01-15||[@ara4n](https://github.com/ara4n)||
|[2967](https://github.com/matrix-org/matrix-spec-proposals/pull/2967)|[WIP] MSC2967: OIDC API scopes|2021-01-14|2024-09-04||[@sandhose](https://github.com/sandhose)||
|[2964](https://github.com/matrix-org/matrix-spec-proposals/pull/2964)|[WIP] MSC2964: Usage of OAuth 2.0 authorization code grant and refresh token grant|2021-01-14|2024-09-04||[@sandhose](https://github.com/sandhose)||
|[2957](https://github.com/matrix-org/matrix-spec-proposals/pull/2957)|[WIP] MSC2957: Cryptographically Concealed Credentials|2021-01-11|2022-03-28||[@uhoreg](https://github.com/uhoreg)||
|[2923](https://github.com/matrix-org/matrix-spec-proposals/pull/2923)|MSC2923: Connect Matrix room to another Matrix room|2020-12-22|2023-06-29||[@MadLittleMods](https://github.com/MadLittleMods)||
|[2883](https://github.com/matrix-org/matrix-spec-proposals/pull/2883)|MSC2883: [WIP] Matrix-flavoured MLS|2020-11-27|2024-07-26||[@uhoreg](https://github.com/uhoreg)||
|[2882](https://github.com/matrix-org/matrix-spec-proposals/pull/2882)|MSC2882: [WIP] Tempered Transitive Trust|2020-11-27|2022-10-05||[@uhoreg](https://github.com/uhoreg)||
|[2812](https://github.com/matrix-org/matrix-spec-proposals/pull/2812)|[WIP] MSC2812: Role-based power/permissions|2020-10-08|2023-09-15||[@turt2live](https://github.com/turt2live)||
|[2802](https://github.com/matrix-org/matrix-spec-proposals/pull/2802)|[WIP] MSC2802: Full Room Abstraction|2020-10-05|2021-08-30||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[2787](https://github.com/matrix-org/matrix-spec-proposals/pull/2787)|MSC2787: Portable Identities|2020-09-23|2024-04-26||[@neilalexander](https://github.com/neilalexander)||
|[2783](https://github.com/matrix-org/matrix-spec-proposals/pull/2783)|MSC2783: Homeserver Migration Data Format|2020-09-19|2024-09-11||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[2745](https://github.com/matrix-org/matrix-spec-proposals/pull/2745)|[WIP] MSC2745 : Add hCaptcha as captcha provider|2020-08-20|2022-03-01||[@Morpheus0x](https://github.com/Morpheus0x)||
|[2706](https://github.com/matrix-org/matrix-spec-proposals/pull/2706)|[WIP] MSC2706: IPFS as a media repository for Matrix|2020-07-28|2023-06-04||[@turt2live](https://github.com/turt2live)||
|[2438](https://github.com/matrix-org/matrix-spec-proposals/pull/2438)|[WIP] MSC2438: Local and Federated User Erasure Requests|2020-02-19|2023-06-02||[@anoadragon453](https://github.com/anoadragon453)||
|[2370](https://github.com/matrix-org/matrix-spec-proposals/pull/2370)|[WIP] MSC2370: Resolve URL API|2019-11-29|2021-08-30||[@uhoreg](https://github.com/uhoreg)||
|[2306](https://github.com/matrix-org/matrix-spec-proposals/pull/2306)|[WIP] MSC2306: Removing MSISDN password resets|2019-09-24|2022-03-01||[@turt2live](https://github.com/turt2live)||
|[2301](https://github.com/matrix-org/matrix-spec-proposals/pull/2301)|[WIP] MSC2301: server info endpoint|2019-09-23|2021-08-30||[@ara4n](https://github.com/ara4n)||
|[2300](https://github.com/matrix-org/matrix-spec-proposals/pull/2300)|[WIP] MSC2300: /ping endpoint|2019-09-23|2021-10-15||[@ara4n](https://github.com/ara4n)||
|[2271](https://github.com/matrix-org/matrix-spec-proposals/pull/2271)|[WIP] MSC2271 TOTP 2FA login|2019-08-31|2021-08-30||[@ara4n](https://github.com/ara4n)||
|[1956](https://github.com/matrix-org/matrix-spec-proposals/pull/1956)|[WIP] MSC1956: Integrations API (base)|2019-04-08|2023-05-02||[@turt2live](https://github.com/turt2live)||
|[1781](https://github.com/matrix-org/matrix-spec-proposals/pull/1781)|[WIP] MSC1781: Proposal for associations for DIDs and DID names|2019-01-07|2022-12-13||[@friedger](https://github.com/friedger)||
|[1780](https://github.com/matrix-org/matrix-spec-proposals/pull/1780)|[WIP] MSC1780: Add DIDs and DID names as admin accounts to HS|2019-01-07|2021-08-30||[@friedger](https://github.com/friedger)||
|[1769](https://github.com/matrix-org/matrix-spec-proposals/pull/1769)|WIPish: MSC1769: Extensible profiles as rooms|2019-01-03|2024-05-29||[@ara4n](https://github.com/ara4n)||
|[1768](https://github.com/matrix-org/matrix-spec-proposals/pull/1768)|[WIP] MSC1768: Proposal to authenticate with public keys|2019-01-03|2021-08-30||[@friedger](https://github.com/friedger)||
|[1762](https://github.com/matrix-org/matrix-spec-proposals/pull/1762)|[WIP] MSC1762: Support user-owned identifiers as new 3PID type|2018-12-28|2021-08-30||[@friedger](https://github.com/friedger)||
|[1607](https://github.com/matrix-org/matrix-spec-proposals/pull/1607)|MSC1607: Proposal for room alias grammar|2018-08-29|2021-08-30||[@richvdh](https://github.com/richvdh)||
|[1228](https://github.com/matrix-org/matrix-spec-proposals/pull/1228)|[WIP] MSC1228: Removing MXIDs from events|2018-05-10|2024-06-05||[@benparsons](https://github.com/benparsons)||

### Proposal In Review[](https://spec.matrix.org/proposals/#proposal-in-review)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4190](https://github.com/matrix-org/matrix-spec-proposals/pull/4190)|MSC4190: Device management for application services|2024-09-12|2024-09-12||[@sandhose](https://github.com/sandhose)||
|[4188](https://github.com/matrix-org/matrix-spec-proposals/pull/4188)|MSC4188: Handling HTTP 410 Gone Status in Matrix Server Discovery|2024-09-04|2024-09-06||[@tcpipuk](https://github.com/tcpipuk)||
|[4186](https://github.com/matrix-org/matrix-spec-proposals/pull/4186)|MSC4186: Simplified Sliding Sync|2024-08-30|2024-09-11||[@erikjohnston](https://github.com/erikjohnston)||
|[4185](https://github.com/matrix-org/matrix-spec-proposals/pull/4185)|MSC4185: Event Visibility API|2024-08-29|2024-08-29||[@tadzik](https://github.com/tadzik)||
|[4184](https://github.com/matrix-org/matrix-spec-proposals/pull/4184)|MSC4184: Dynamic Notification Suppression|2024-08-28|2024-08-30||[@turt2live](https://github.com/turt2live)||
|[4177](https://github.com/matrix-org/matrix-spec-proposals/pull/4177)|MSC4177: Add upload location hints proposal|2024-08-13|2024-08-26||[@Fizzadar](https://github.com/Fizzadar)||
|[4176](https://github.com/matrix-org/matrix-spec-proposals/pull/4176)|MSC4176: Translatable Errors|2024-08-13|2024-08-14||[@turt2live](https://github.com/turt2live)||
|[4175](https://github.com/matrix-org/matrix-spec-proposals/pull/4175)|MSC4175: Profile field for user time zone|2024-08-05|2024-09-10||[@clokep](https://github.com/clokep)||
|[4174](https://github.com/matrix-org/matrix-spec-proposals/pull/4174)|MSC4174: webpush push kind|2024-08-01|2024-09-13||[@p1gp1g](https://github.com/p1gp1g)||
|[4173](https://github.com/matrix-org/matrix-spec-proposals/pull/4173)|MSC4173: test pusher|2024-07-31|2024-08-01||[@p1gp1g](https://github.com/p1gp1g)||
|[4171](https://github.com/matrix-org/matrix-spec-proposals/pull/4171)|MSC4171: Service members|2024-07-19|2024-08-07||[@tulir](https://github.com/tulir)||
|[4168](https://github.com/matrix-org/matrix-spec-proposals/pull/4168)|MSC4168: Update `m.space.*` state on room upgrade|2024-07-06|2024-07-12||[@Kladki](https://github.com/Kladki)||
|[4167](https://github.com/matrix-org/matrix-spec-proposals/pull/4167)|MSC4167: Copy bans on room upgrade|2024-07-06|2024-07-12||[@Kladki](https://github.com/Kladki)||
|[4166](https://github.com/matrix-org/matrix-spec-proposals/pull/4166)|MSC4166: Specify `/turnServer` response when no TURN servers are available|2024-07-06|2024-09-02||[@Kladki](https://github.com/Kladki)||
|[4165](https://github.com/matrix-org/matrix-spec-proposals/pull/4165)|MSC4165: Remove own power level on deactivation|2024-07-06|2024-07-08||[@Kladki](https://github.com/Kladki)||
|[4164](https://github.com/matrix-org/matrix-spec-proposals/pull/4164)|MSC4164: Leave all rooms on deactivation|2024-07-06|2024-08-28||[@Kladki](https://github.com/Kladki)||
|[4162](https://github.com/matrix-org/matrix-spec-proposals/pull/4162)|MSC4162: One-Time Key Reset Endpoint|2024-07-02|2024-07-09||[@kegsay](https://github.com/kegsay)||
|[4161](https://github.com/matrix-org/matrix-spec-proposals/pull/4161)|MSC4161: Crypto terminology for non-technical users|2024-06-27|2024-09-09||[@andybalaam](https://github.com/andybalaam)||
|[4158](https://github.com/matrix-org/matrix-spec-proposals/pull/4158)|MSC4158: MatrixRTC focus information in .well-known|2024-06-20|2024-09-11||[@toger5](https://github.com/toger5)||
|[4155](https://github.com/matrix-org/matrix-spec-proposals/pull/4155)|MSC4155: Invite filtering|2024-06-13|2024-06-13||[@Johennes](https://github.com/Johennes)||
|[4154](https://github.com/matrix-org/matrix-spec-proposals/pull/4154)|MSC4154: Request max body size|2024-06-11|2024-06-15||[@turt2live](https://github.com/turt2live)||
|[4153](https://github.com/matrix-org/matrix-spec-proposals/pull/4153)|MSC4153: Exclude non-cross-signed devices|2024-06-10|2024-09-10||[@uhoreg](https://github.com/uhoreg)||
|[4150](https://github.com/matrix-org/matrix-spec-proposals/pull/4150)|MSC4150: m.allow recommendation for moderation policy lists|2024-06-04|2024-07-24||[@Johennes](https://github.com/Johennes)||
|[4149](https://github.com/matrix-org/matrix-spec-proposals/pull/4149)|MSC4149: Update CSP Directives for Media Repository|2024-06-04|2024-09-04||[@tcpipuk](https://github.com/tcpipuk)||
|[4147](https://github.com/matrix-org/matrix-spec-proposals/pull/4147)|MSC4147: Including device keys with Olm-encrypted events|2024-05-29|2024-08-09||[@uhoreg](https://github.com/uhoreg)||
|[4146](https://github.com/matrix-org/matrix-spec-proposals/pull/4146)|MSC4146: Shared Message Drafts|2024-05-23|2024-07-31||[@cyrneko](https://github.com/cyrneko)||
|[4144](https://github.com/matrix-org/matrix-spec-proposals/pull/4144)|MSC4144: Per-message profiles|2024-05-10|2024-07-29||[@tulir](https://github.com/tulir)||
|[4142](https://github.com/matrix-org/matrix-spec-proposals/pull/4142)|MSC4142: Remove unintentional intentional mentions in replies|2024-05-09|2024-05-10||[@tulir](https://github.com/tulir)||
|[4141](https://github.com/matrix-org/matrix-spec-proposals/pull/4141)|MSC4141: Time based notification filtering|2024-05-08|2024-06-22||[@hanadi92](https://github.com/hanadi92)||
|[4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140)|MSC4140: Cancellable delayed events|2024-05-07|2024-09-11||[@toger5](https://github.com/toger5)||
|[4136](https://github.com/matrix-org/matrix-spec-proposals/pull/4136)|MSC4136: Shared retry hints|2024-04-26|2024-04-30||[@ara4n](https://github.com/ara4n)||
|[4131](https://github.com/matrix-org/matrix-spec-proposals/pull/4131)|MSC4131: Handling `m.room.encryption` events|2024-04-17|2024-04-23||[@uhoreg](https://github.com/uhoreg)||
|[4128](https://github.com/matrix-org/matrix-spec-proposals/pull/4128)|MSC4128: Error on invalid auth where it is optional|2024-04-11|2024-04-21||[@Kladki](https://github.com/Kladki)||
|[4127](https://github.com/matrix-org/matrix-spec-proposals/pull/4127)|MSC4127: Removal of query string auth|2024-04-10|2024-04-12||[@turt2live](https://github.com/turt2live)||
|[4125](https://github.com/matrix-org/matrix-spec-proposals/pull/4125)|MSC4125: Specify servers to join via for federated invites|2024-04-09|2024-04-10||[@Kladki](https://github.com/Kladki)||
|[4124](https://github.com/matrix-org/matrix-spec-proposals/pull/4124)|MSC4124: Simple Server Authorization|2024-04-09|2024-04-14||[@Gnuxie](https://github.com/Gnuxie)||
|[4121](https://github.com/matrix-org/matrix-spec-proposals/pull/4121)|MSC4121: `m.role.moderator` `/.well-known/matrix/support` role.|2024-03-22|2024-06-01||[@FSG-Cat](https://github.com/FSG-Cat)||
|[4120](https://github.com/matrix-org/matrix-spec-proposals/pull/4120)|MSC4120: Allow `HEAD` on `/download`|2024-03-21|2024-05-03||[@turt2live](https://github.com/turt2live)||
|[4119](https://github.com/matrix-org/matrix-spec-proposals/pull/4119)|MSC4119: Voluntary content flagging|2024-03-13|2024-09-01||[@turt2live](https://github.com/turt2live)||
|[4117](https://github.com/matrix-org/matrix-spec-proposals/pull/4117)|MSC4117: Reinstating Events (Reversible Redactions)|2024-03-04|2024-03-04||[@turt2live](https://github.com/turt2live)||
|[4114](https://github.com/matrix-org/matrix-spec-proposals/pull/4114)|MSC4114: Matrix as a password manager|2024-02-24|2024-06-06||[@Johennes](https://github.com/Johennes)||
|[4113](https://github.com/matrix-org/matrix-spec-proposals/pull/4113)|MSC4113: Image hashes in Policy Lists|2024-02-24|2024-02-24||[@MTRNord](https://github.com/MTRNord)||
|[4110](https://github.com/matrix-org/matrix-spec-proposals/pull/4110)|MSC4110: Fewer Features|2024-02-23|2024-03-15||[@cloudrac3r](https://github.com/cloudrac3r)||
|[4108](https://github.com/matrix-org/matrix-spec-proposals/pull/4108)|MSC4108: Mechanism to allow OIDC sign in and E2EE set up via QR code|2024-02-22|2024-07-08||[@hughns](https://github.com/hughns)||
|[4103](https://github.com/matrix-org/matrix-spec-proposals/pull/4103)|MSC4103: Make threaded read receipts opt-in in /sync|2024-02-15|2024-02-20||[@ara4n](https://github.com/ara4n)||
|[4102](https://github.com/matrix-org/matrix-spec-proposals/pull/4102)|MSC4102: Clarifying precedence in threaded and unthreaded read receipts in EDUs|2024-02-15|2024-02-16||[@kegsay](https://github.com/kegsay)||
|[4098](https://github.com/matrix-org/matrix-spec-proposals/pull/4098)|MSC4098: Use the SCIM protocol for provisioning|2024-02-09|2024-08-15||[@azmeuk](https://github.com/azmeuk)||
|[4096](https://github.com/matrix-org/matrix-spec-proposals/pull/4096)|MSC4096: Proposal to make forceTurn option configurable server-side|2024-02-06|2024-02-06||[@raphaelbadawi](https://github.com/raphaelbadawi)||
|[4095](https://github.com/matrix-org/matrix-spec-proposals/pull/4095)|MSC4095: Bundled URL previews|2024-01-31|2024-08-19||[@tulir](https://github.com/tulir)||
|[4092](https://github.com/matrix-org/matrix-spec-proposals/pull/4092)|MSC4092: Enforce tests around sensitive parts of the specification|2024-01-23|2024-01-24||[@kegsay](https://github.com/kegsay)||
|[4084](https://github.com/matrix-org/matrix-spec-proposals/pull/4084)|MSC4084: Improving security of MSC2244 (Mass redactions)|2023-12-18|2024-03-09||[@turt2live](https://github.com/turt2live)||
|[4083](https://github.com/matrix-org/matrix-spec-proposals/pull/4083)|MSC4083: Delta-compressed E2EE file transfers|2023-12-16|2023-12-27||[@ara4n](https://github.com/ara4n)||
|[4081](https://github.com/matrix-org/matrix-spec-proposals/pull/4081)|MSC4081: Eagerly sharing fallback keys with federated servers|2023-11-21|2024-07-12||[@kegsay](https://github.com/kegsay)||
|[4079](https://github.com/matrix-org/matrix-spec-proposals/pull/4079)|MSC4079: Server-Defined Client Landing Pages|2023-11-14|2023-11-21||[@williamkray](https://github.com/williamkray)||
|[4078](https://github.com/matrix-org/matrix-spec-proposals/pull/4078)|MSC4078: Transparent pusher creation|2023-11-14|2023-12-21||[@stefanceriu](https://github.com/stefanceriu)||
|[4076](https://github.com/matrix-org/matrix-spec-proposals/pull/4076)|MSC4076: Let E2EE clients calculate app & E2EE-room badge counts themselves|2023-11-12|2024-09-03||[@ara4n](https://github.com/ara4n)||
|[4075](https://github.com/matrix-org/matrix-spec-proposals/pull/4075)|MSC4075: MatrixRTC Call Ringing|2023-11-08|2024-07-10||[@toger5](https://github.com/toger5)||
|[4074](https://github.com/matrix-org/matrix-spec-proposals/pull/4074)|MSC4074: Server side annotation aggregation|2023-11-07|2024-06-27||[@dihydrogenmonoxide16](https://github.com/dihydrogenmonoxide16)||
|[4073](https://github.com/matrix-org/matrix-spec-proposals/pull/4073)|MSC4073: Shepherd teams|2023-11-07|2024-05-13||[@joepie91](https://github.com/joepie91)||
|[4071](https://github.com/matrix-org/matrix-spec-proposals/pull/4071)|MSC4071: Pagination Token Headers|2023-10-29|2024-01-24||[@Fizzadar](https://github.com/Fizzadar)||
|[4069](https://github.com/matrix-org/matrix-spec-proposals/pull/4069)|MSC4069: Inhibit profile propagation|2023-10-23|2023-11-15||[@turt2live](https://github.com/turt2live)||
|[4062](https://github.com/matrix-org/matrix-spec-proposals/pull/4062)|MSC4062: Add a new push rule tweak to disable email notification|2023-10-20|2023-10-23||[@giomfo](https://github.com/giomfo)||
|[4053](https://github.com/matrix-org/matrix-spec-proposals/pull/4053)|MSC4053: Extensible Events - Mentions mixin|2023-09-07|2023-09-08||[@clokep](https://github.com/clokep)||
|[4050](https://github.com/matrix-org/matrix-spec-proposals/pull/4050)|MSC4050: MXID verification|2023-09-02|2024-01-24||[@jucktnich](https://github.com/jucktnich)||
|[4048](https://github.com/matrix-org/matrix-spec-proposals/pull/4048)|MSC4048: Authenticated key backup|2023-08-23|2023-11-23||[@uhoreg](https://github.com/uhoreg)||
|[4045](https://github.com/matrix-org/matrix-spec-proposals/pull/4045)|MSC4045: Deprecating the use of IP addresses in server names|2023-08-12|2023-09-27||[@turt2live](https://github.com/turt2live)||
|[4043](https://github.com/matrix-org/matrix-spec-proposals/pull/4043)|MSC4043: Presence Override API|2023-08-10|2024-02-19||[@FSG-Cat](https://github.com/FSG-Cat)||
|[4042](https://github.com/matrix-org/matrix-spec-proposals/pull/4042)|MSC4042: Disabled Presence State|2023-08-10|2023-09-15||[@FSG-Cat](https://github.com/FSG-Cat)||
|[4039](https://github.com/matrix-org/matrix-spec-proposals/pull/4039)|MSC4039: Access the Content repository with the Widget API|2023-07-24|2024-08-30||[@dhenneke](https://github.com/dhenneke)||
|[4036](https://github.com/matrix-org/matrix-spec-proposals/pull/4036)|MSC4036: Room organization by promoting threads|2023-07-13|2023-10-17||[@C0ffeeCode](https://github.com/C0ffeeCode)||
|[4034](https://github.com/matrix-org/matrix-spec-proposals/pull/4034)|MSC4034: Media limits|2023-07-10|2023-07-18||[@gitsper](https://github.com/gitsper)||
|[4033](https://github.com/matrix-org/matrix-spec-proposals/pull/4033)|MSC4033: Explicit ordering of events for receipts|2023-07-04|2024-02-13||[@andybalaam](https://github.com/andybalaam)||
|[4032](https://github.com/matrix-org/matrix-spec-proposals/pull/4032)|MSC4032: Asset Collections|2023-06-28|2023-11-27||[@robertlong](https://github.com/robertlong)||
|[4030](https://github.com/matrix-org/matrix-spec-proposals/pull/4030)|MSC4030: Progressive image in Matrix|2023-06-14|2024-08-28||[@ru-ka](https://github.com/ru-ka)||
|[4028](https://github.com/matrix-org/matrix-spec-proposals/pull/4028)|MSC4028: Push all encrypted events except for muted rooms|2023-06-13|2024-01-10||[@giomfo](https://github.com/giomfo)||
|[4027](https://github.com/matrix-org/matrix-spec-proposals/pull/4027)|MSC4027: Propose method of specifying custom images in reactions|2023-06-10|2024-04-15||[@sumnerevans](https://github.com/sumnerevans)||
|[4023](https://github.com/matrix-org/matrix-spec-proposals/pull/4023)|MSC4023: Thread ID for 2nd order-relation|2023-05-31|2023-12-06||[@germain-gg](https://github.com/germain-gg)||
|[4020](https://github.com/matrix-org/matrix-spec-proposals/pull/4020)|MSC4020: Room model configuration|2023-05-25|2023-05-25||[@turt2live](https://github.com/turt2live)||
|[4019](https://github.com/matrix-org/matrix-spec-proposals/pull/4019)|MSC4019: Encrypted event relationships|2023-05-20|2024-07-05||[@tusooa](https://github.com/tusooa)||
|[4016](https://github.com/matrix-org/matrix-spec-proposals/pull/4016)|MSC4016: Streaming E2EE file transfers with random access|2023-05-14|2024-07-26||[@ara4n](https://github.com/ara4n)||
|[4015](https://github.com/matrix-org/matrix-spec-proposals/pull/4015)|MSC4015: Voluntary Bot indicators|2023-05-14|2024-01-24||[@MTRNord](https://github.com/MTRNord)||
|[4014](https://github.com/matrix-org/matrix-spec-proposals/pull/4014)|MSC4014: Pseudonymous Identities|2023-05-10|2023-11-14||[@kegsay](https://github.com/kegsay)||
|[4013](https://github.com/matrix-org/matrix-spec-proposals/pull/4013)|MSC4013: Poll history cache|2023-05-10|2023-11-29||[@alfogrillo](https://github.com/alfogrillo)||
|[4006](https://github.com/matrix-org/matrix-spec-proposals/pull/4006)|MSC4006: "completed elsewhere" hangup reason.|2023-05-02|2023-05-05||[@dbkr](https://github.com/dbkr)||
|[4005](https://github.com/matrix-org/matrix-spec-proposals/pull/4005)|MSC4005: Explicit read receipts for sent events|2023-05-01|2023-08-04||[@bradtgmurray](https://github.com/bradtgmurray)||
|[4004](https://github.com/matrix-org/matrix-spec-proposals/pull/4004)|MSC4004: unified view of identity service|2023-04-26|2023-12-06||[@guimard](https://github.com/guimard)||
|[4003](https://github.com/matrix-org/matrix-spec-proposals/pull/4003)|MSC4003: Semantic table attributes|2023-04-25|2024-01-12||[@AndrewKvalheim](https://github.com/AndrewKvalheim)||
|[4001](https://github.com/matrix-org/matrix-spec-proposals/pull/4001)|MSC4001: Return start of room state at context endpoint|2023-04-21|2023-04-26||[@BramvdnHeuvel](https://github.com/BramvdnHeuvel)||
|[4000](https://github.com/matrix-org/matrix-spec-proposals/pull/4000)|MSC4000: Forwards fill (`/backfill` forwards)|2023-04-14|2023-04-19||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3999](https://github.com/matrix-org/matrix-spec-proposals/pull/3999)|MSC3999: Add causal parameter to `/timestamp_to_event`|2023-04-14|2024-03-16||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3998](https://github.com/matrix-org/matrix-spec-proposals/pull/3998)|MSC3998: Add timestamp massaging to `/join` and `/knock`|2023-04-13|2023-04-27||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3997](https://github.com/matrix-org/matrix-spec-proposals/pull/3997)|MSC3997: Add timestamp massaging to `/createRoom`|2023-04-13|2023-04-13||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3996](https://github.com/matrix-org/matrix-spec-proposals/pull/3996)|MSC3996: Encrypted mentions-only rooms.|2023-04-13|2023-06-22||[@clokep](https://github.com/clokep)||
|[3993](https://github.com/matrix-org/matrix-spec-proposals/pull/3993)|MSC3993: Room takeover|2023-04-05|2024-05-09||[@jaller94](https://github.com/jaller94)||
|[3991](https://github.com/matrix-org/matrix-spec-proposals/pull/3991)|MSC3991: Power level up! Taking the room to new heights|2023-04-04|2023-04-14||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3985](https://github.com/matrix-org/matrix-spec-proposals/pull/3985)|MSC3985: Break-out rooms|2023-03-27|2024-02-22||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3984](https://github.com/matrix-org/matrix-spec-proposals/pull/3984)|MSC3984: Sending key queries to appservices|2023-03-24|2023-06-01||[@turt2live](https://github.com/turt2live)||
|[3983](https://github.com/matrix-org/matrix-spec-proposals/pull/3983)|MSC3983: Sending One-Time Key (OTK) claims to appservices|2023-03-23|2023-09-20||[@turt2live](https://github.com/turt2live)||
|[3982](https://github.com/matrix-org/matrix-spec-proposals/pull/3982)|MSC3982: Limit maximum number of events sent to an AS|2023-03-21|2023-04-03||[@Half-Shot](https://github.com/Half-Shot)||
|[3979](https://github.com/matrix-org/matrix-spec-proposals/pull/3979)|MSC3979: Revised feature profiles|2023-03-15|2023-03-16||[@uhoreg](https://github.com/uhoreg)||
|[3977](https://github.com/matrix-org/matrix-spec-proposals/pull/3977)|MSC3977: Matrix as a Messaging Framework (IETF/MIMI)|2023-03-13|2023-03-13||[@turt2live](https://github.com/turt2live)||
|[3975](https://github.com/matrix-org/matrix-spec-proposals/pull/3975)|MSC3975: rel_type for Replies|2023-03-08|2023-03-27||[@imbev](https://github.com/imbev)||
|[3973](https://github.com/matrix-org/matrix-spec-proposals/pull/3973)|MSC3973: Search users in the user directory with the Widget API|2023-03-01|2023-03-16||[@dhenneke](https://github.com/dhenneke)||
|[3972](https://github.com/matrix-org/matrix-spec-proposals/pull/3972)|MSC3972: Lexicographical strings as an ordering mechanism|2023-02-26|2023-03-02||[@Dominaezzz](https://github.com/Dominaezzz)||
|[3971](https://github.com/matrix-org/matrix-spec-proposals/pull/3971)|MSC3971: Sharing image packs|2023-02-25|2023-03-11||[@AndrewRyanChama](https://github.com/AndrewRyanChama)||
|[3963](https://github.com/matrix-org/matrix-spec-proposals/pull/3963)|MSC3963: Oblivious Matrix over HTTPS|2023-02-03|2023-11-22||[@ghost-amnesiac](https://github.com/ghost-amnesiac)||
|[3961](https://github.com/matrix-org/matrix-spec-proposals/pull/3961)|MSC3961: Sliding Sync Extension: Typing Notifications|2023-01-30|2023-03-31||[@kegsay](https://github.com/kegsay)||
|[3960](https://github.com/matrix-org/matrix-spec-proposals/pull/3960)|MSC3960: Sliding Sync Extension: Receipts|2023-01-30|2024-08-06||[@kegsay](https://github.com/kegsay)||
|[3959](https://github.com/matrix-org/matrix-spec-proposals/pull/3959)|MSC3959: Sliding Sync Extension: Account Data|2023-01-30|2023-06-06||[@kegsay](https://github.com/kegsay)||
|[3955](https://github.com/matrix-org/matrix-spec-proposals/pull/3955)|MSC3955: Extensible Events - Automated event mixin (notices)|2023-01-13|2023-11-30||[@turt2live](https://github.com/turt2live)||
|[3954](https://github.com/matrix-org/matrix-spec-proposals/pull/3954)|MSC3954: Extensible Events - Text Emotes|2023-01-13|2023-11-29||[@turt2live](https://github.com/turt2live)||
|[3949](https://github.com/matrix-org/matrix-spec-proposals/pull/3949)|MSC3949: Power Level Tags|2022-12-14|2023-04-23||[@ajbura](https://github.com/ajbura)||
|[3947](https://github.com/matrix-org/matrix-spec-proposals/pull/3947)|MSC3947: Allow Clients to Request Searching the User Directory Constrained to Only Homeserver-Local Users|2022-12-11|2022-12-14||[@marcusmueller](https://github.com/marcusmueller)||
|[3946](https://github.com/matrix-org/matrix-spec-proposals/pull/3946)|MSC3946: Dynamic room predecessor|2022-12-09|2023-04-12||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3934](https://github.com/matrix-org/matrix-spec-proposals/pull/3934)|MSC3934: Bulk push rules change endpoint|2022-11-15|2022-12-12||[@turt2live](https://github.com/turt2live)||
|[3933](https://github.com/matrix-org/matrix-spec-proposals/pull/3933)|MSC3933: Core push rules for Extensible Events|2022-11-15|2023-01-17||[@turt2live](https://github.com/turt2live)||
|[3932](https://github.com/matrix-org/matrix-spec-proposals/pull/3932)|MSC3932: Extensible events room version push rule feature flag|2022-11-15|2023-02-28||[@turt2live](https://github.com/turt2live)||
|[3931](https://github.com/matrix-org/matrix-spec-proposals/pull/3931)|MSC3931: Push rule condition for room version features|2022-11-15|2022-11-15||[@turt2live](https://github.com/turt2live)||
|[3927](https://github.com/matrix-org/matrix-spec-proposals/pull/3927)|MSC3927: Extensible Events - Audio|2022-11-13|2023-04-18||[@turt2live](https://github.com/turt2live)||
|[3926](https://github.com/matrix-org/matrix-spec-proposals/pull/3926)|MSC3926: Disable server-default notifications for bot users by default|2022-11-08|2022-11-10||[@Half-Shot](https://github.com/Half-Shot)||
|[3922](https://github.com/matrix-org/matrix-spec-proposals/pull/3922)|MSC3922: Removing SRV records from homeserver discovery|2022-11-01|2024-07-07||[@turt2live](https://github.com/turt2live)||
|[3919](https://github.com/matrix-org/matrix-spec-proposals/pull/3919)|MSC3919: Matrix Message Format (IETF/MIMI)|2022-10-28|2022-10-28||[@turt2live](https://github.com/turt2live)||
|[3918](https://github.com/matrix-org/matrix-spec-proposals/pull/3918)|MSC3918: Matrix Message Transport (IETF/MIMI)|2022-10-28|2022-10-28||[@turt2live](https://github.com/turt2live)||
|[3917](https://github.com/matrix-org/matrix-spec-proposals/pull/3917)|MSC3917: Cryptographically Constrained Room Membership|2022-10-25|2024-06-21||[@duxovni](https://github.com/duxovni)||
|[3915](https://github.com/matrix-org/matrix-spec-proposals/pull/3915)|MSC3915: Owner power level|2022-10-23|2023-06-06||[@ara4n](https://github.com/ara4n)||
|[3914](https://github.com/matrix-org/matrix-spec-proposals/pull/3914)|MSC3914: Matrix native group call push rule|2022-10-20|2023-05-12||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3912](https://github.com/matrix-org/matrix-spec-proposals/pull/3912)|MSC3912: Redaction of related events|2022-10-18|2023-06-26||[@babolivier](https://github.com/babolivier)||
|[3911](https://github.com/matrix-org/matrix-spec-proposals/pull/3911)|MSC3911: Linking media to events|2022-10-16|2024-08-16||[@richvdh](https://github.com/richvdh)||
|[3908](https://github.com/matrix-org/matrix-spec-proposals/pull/3908)|MSC3908: Expiring Policy List Recommendations|2022-10-13|2024-01-30||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3907](https://github.com/matrix-org/matrix-spec-proposals/pull/3907)|MSC3907: Mute Policy Recommendation|2022-10-13|2024-05-20||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3896](https://github.com/matrix-org/matrix-spec-proposals/pull/3896)|MSC3896: Appservice media|2022-09-24|2024-07-01||[@tezlm](https://github.com/tezlm)||
|[3892](https://github.com/matrix-org/matrix-spec-proposals/pull/3892)|MSC3892: Custom Emotes with Encryption|2022-09-14|2023-10-24||[@nmscode](https://github.com/nmscode)||
|[3890](https://github.com/matrix-org/matrix-spec-proposals/pull/3890)|MSC3890: Remotely silence local notifications|2022-09-13|2023-02-20||[@kerryarchibald](https://github.com/kerryarchibald)||
|[3888](https://github.com/matrix-org/matrix-spec-proposals/pull/3888)|MSC3888: Voice Broadcast|2022-09-09|2023-06-05||[@weeman1337](https://github.com/weeman1337)||
|[3885](https://github.com/matrix-org/matrix-spec-proposals/pull/3885)|MSC3885: Sliding Sync Extension: To-Device messages|2022-09-07|2024-07-24||[@kegsay](https://github.com/kegsay)||
|[3884](https://github.com/matrix-org/matrix-spec-proposals/pull/3884)|MSC3884: Sliding Sync Extension: E2EE|2022-09-06|2024-01-04||[@kegsay](https://github.com/kegsay)||
|[3881](https://github.com/matrix-org/matrix-spec-proposals/pull/3881)|MSC3881: Remotely toggle push notifications for another client|2022-09-05|2022-11-21||[@kerryarchibald](https://github.com/kerryarchibald)||
|[3880](https://github.com/matrix-org/matrix-spec-proposals/pull/3880)|MSC3880: dummy replies for Olm|2022-09-05|2022-09-07||[@uhoreg](https://github.com/uhoreg)||
|[3879](https://github.com/matrix-org/matrix-spec-proposals/pull/3879)|MSC3879: Trusted key forwards|2022-09-05|2023-10-16||[@uhoreg](https://github.com/uhoreg)||
|[3874](https://github.com/matrix-org/matrix-spec-proposals/pull/3874)|MSC3874: Filtering threads from the /messages endpoint|2022-08-23|2023-06-06||[@justjanne](https://github.com/justjanne)||
|[3871](https://github.com/matrix-org/matrix-spec-proposals/pull/3871)|MSC3871: Gappy timelines|2022-08-18|2024-05-29||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3870](https://github.com/matrix-org/matrix-spec-proposals/pull/3870)|MSC3870: Async media upload extension: upload to URL|2022-08-17|2024-08-10||[@Fizzadar](https://github.com/Fizzadar)||
|[3869](https://github.com/matrix-org/matrix-spec-proposals/pull/3869)|MSC3869: Read event relations with the Widget API|2022-08-16|2022-11-09||[@dhenneke](https://github.com/dhenneke)||
|[3866](https://github.com/matrix-org/matrix-spec-proposals/pull/3866)|MSC3866: `M_USER_AWAITING_APPROVAL` error code|2022-08-12|2022-09-29||[@babolivier](https://github.com/babolivier)||
|[3857](https://github.com/matrix-org/matrix-spec-proposals/pull/3857)|MSC3857: Welcome messages|2022-08-02|2022-08-14||[@turt2live](https://github.com/turt2live)||
|[3851](https://github.com/matrix-org/matrix-spec-proposals/pull/3851)|MSC3851: Allow custom room presets when creating a room|2022-07-21|2022-07-22||[@anoadragon453](https://github.com/anoadragon453)||
|[3848](https://github.com/matrix-org/matrix-spec-proposals/pull/3848)|MSC3848: Introduce errcodes for specific event sending failures|2022-07-19|2022-10-11||[@Half-Shot](https://github.com/Half-Shot)||
|[3847](https://github.com/matrix-org/matrix-spec-proposals/pull/3847)|MSC3847: Ignoring invites with policy rooms|2022-07-13|2024-06-02||[@Yoric](https://github.com/Yoric)||
|[3846](https://github.com/matrix-org/matrix-spec-proposals/pull/3846)|MSC3846: Allowing widgets to access TURN servers|2022-07-13|2022-08-01||[@robintown](https://github.com/robintown)||
|[3845](https://github.com/matrix-org/matrix-spec-proposals/pull/3845)|MSC3845: Draft: Expanding policy rooms to reputation|2022-07-12|2022-12-22||[@Yoric](https://github.com/Yoric)||
|[3843](https://github.com/matrix-org/matrix-spec-proposals/pull/3843)|MSC3843: Reporting content over federation|2022-07-11|2024-06-12||[@turt2live](https://github.com/turt2live)||
|[3840](https://github.com/matrix-org/matrix-spec-proposals/pull/3840)|MSC3840: Ignore invites|2022-06-28|2024-05-31||[@Yoric](https://github.com/Yoric)||
|[3837](https://github.com/matrix-org/matrix-spec-proposals/pull/3837)|MSC3837: Cascading profile tags for push rules|2022-06-23|2024-06-06||[@Johennes](https://github.com/Johennes)||
|[3834](https://github.com/matrix-org/matrix-spec-proposals/pull/3834)|MSC3834: Opportunistic user key pinning (TOFU)|2022-06-14|2023-05-11||[@duxovni](https://github.com/duxovni)||
|[3825](https://github.com/matrix-org/matrix-spec-proposals/pull/3825)|MSC3825: Obvious relation fallback location|2022-05-24|2022-10-12||[@deepbluev7](https://github.com/deepbluev7)||
|[3824](https://github.com/matrix-org/matrix-spec-proposals/pull/3824)|MSC3824: OIDC aware clients|2022-05-24|2024-07-08||[@hughns](https://github.com/hughns)||
|[3823](https://github.com/matrix-org/matrix-spec-proposals/pull/3823)|MSC3823: Account Suspension|2022-05-23|2024-05-01||[@Yoric](https://github.com/Yoric)||
|[3819](https://github.com/matrix-org/matrix-spec-proposals/pull/3819)|MSC3819: Allowing widgets to send/receive to-device messages|2022-05-19|2023-03-23||[@turt2live](https://github.com/turt2live)||
|[3817](https://github.com/matrix-org/matrix-spec-proposals/pull/3817)|MSC3817: Allow widgets to create rooms|2022-05-17|2022-08-05||[@dhenneke](https://github.com/dhenneke)||
|[3784](https://github.com/matrix-org/matrix-spec-proposals/pull/3784)|MSC3784: Using room type of `m.policy` for policy rooms|2022-04-25|2024-06-13||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3782](https://github.com/matrix-org/matrix-spec-proposals/pull/3782)|MSC3782: Matrix public key login spec|2022-04-21|2022-10-21||[@tak-hntlabs](https://github.com/tak-hntlabs)||
|[3780](https://github.com/matrix-org/matrix-spec-proposals/pull/3780)|MSC3780: Knocking on `action=join`|2022-04-20|2022-08-05||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3779](https://github.com/matrix-org/matrix-spec-proposals/pull/3779)|MSC3779: "Owned" State Events|2022-04-20|2024-07-25||[@andybalaam](https://github.com/andybalaam)||
|[3768](https://github.com/matrix-org/matrix-spec-proposals/pull/3768)|MSC3768: Push rule action for in-app notifications|2022-04-08|2024-06-06||[@Johennes](https://github.com/Johennes)||
|[3767](https://github.com/matrix-org/matrix-spec-proposals/pull/3767)|MSC3767: Time based notification filtering|2022-04-07|2024-02-27||[@kerryarchibald](https://github.com/kerryarchibald)||
|[3765](https://github.com/matrix-org/matrix-spec-proposals/pull/3765)|MSC3765: Rich text in room topics|2022-04-03|2024-09-03||[@Johennes](https://github.com/Johennes)||
|[3759](https://github.com/matrix-org/matrix-spec-proposals/pull/3759)|MSC3759: Leave event metadata for deactivated users|2022-03-31|2022-04-24||[@Half-Shot](https://github.com/Half-Shot)||
|[3754](https://github.com/matrix-org/matrix-spec-proposals/pull/3754)|MSC3754: Removing profile information|2022-03-12|2024-08-23||[@zecakeh](https://github.com/zecakeh)||
|[3751](https://github.com/matrix-org/matrix-spec-proposals/pull/3751)|MSC3751: Allowing widgets to read account data|2022-03-05|2022-05-05||[@AndrewRyanChama](https://github.com/AndrewRyanChama)||
|[3744](https://github.com/matrix-org/matrix-spec-proposals/pull/3744)|MSC3744: Support for flexible authentication|2022-02-24|2022-05-05||[@cvwright](https://github.com/cvwright)||
|[3741](https://github.com/matrix-org/matrix-spec-proposals/pull/3741)|MSC3741: Revealing the useful login flows to clients after a soft logout|2022-02-23|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[3735](https://github.com/matrix-org/matrix-spec-proposals/pull/3735)|MSC3735: Add device information to m.room_key.withheld message|2022-02-18|2024-07-05||[@uhoreg](https://github.com/uhoreg)||
|[3725](https://github.com/matrix-org/matrix-spec-proposals/pull/3725)|MSC3725: Content warnings|2022-02-14|2024-06-17||[@robintown](https://github.com/robintown)||
|[3723](https://github.com/matrix-org/matrix-spec-proposals/pull/3723)|MSC3723: Federation `/versions`|2022-02-11|2024-06-24||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3720](https://github.com/matrix-org/matrix-spec-proposals/pull/3720)|MSC3720: Account status|2022-02-09|2022-09-01||[@babolivier](https://github.com/babolivier)||
|[3713](https://github.com/matrix-org/matrix-spec-proposals/pull/3713)|MSC3713: Alleviating ACL exhaustion with ACL Slots|2022-02-06|2022-05-05||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3682](https://github.com/matrix-org/matrix-spec-proposals/pull/3682)|MSC3682: Sending Account Data to Application Services|2022-01-28|2024-06-04||[@reivilibre](https://github.com/reivilibre)||
|[3673](https://github.com/matrix-org/matrix-spec-proposals/pull/3673)|MSC3673: Encrypted ephemeral data units|2022-01-25|2022-05-06||[@stefanceriu](https://github.com/stefanceriu)|[@anoadragon453](https://github.com/anoadragon453)|
|[3672](https://github.com/matrix-org/matrix-spec-proposals/pull/3672)|MSC3672: Sharing ephemeral streams of location data|2022-01-25|2022-05-09||[@stefanceriu](https://github.com/stefanceriu)|[@anoadragon453](https://github.com/anoadragon453)|
|[3664](https://github.com/matrix-org/matrix-spec-proposals/pull/3664)|MSC3664: Pushrules for relations|2022-01-21|2022-12-07||[@deepbluev7](https://github.com/deepbluev7)||
|[3659](https://github.com/matrix-org/matrix-spec-proposals/pull/3659)|MSC3659: Invite Rules|2022-01-19|2023-06-01||[@joshqou](https://github.com/joshqou)||
|[3644](https://github.com/matrix-org/matrix-spec-proposals/pull/3644)|MSC3644: Extensible Events: Edits and replies|2022-01-14|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[3635](https://github.com/matrix-org/matrix-spec-proposals/pull/3635)|MSC3635: Early Media for VoIP|2022-01-10|2022-05-05||[@dbkr](https://github.com/dbkr)||
|[3618](https://github.com/matrix-org/matrix-spec-proposals/pull/3618)|MSC3618: Add proposal to simplify federation `/send` response|2022-01-04|2022-05-05||[@neilalexander](https://github.com/neilalexander)||
|[3593](https://github.com/matrix-org/matrix-spec-proposals/pull/3593)|MSC3593: Safety Controls through a generic Administration API|2021-12-25|2022-05-16||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3575](https://github.com/matrix-org/matrix-spec-proposals/pull/3575)|MSC3575: Sliding Sync (aka Sync v3)|2021-12-20|2024-09-05||[@kegsay](https://github.com/kegsay)||
|[3574](https://github.com/matrix-org/matrix-spec-proposals/pull/3574)|MSC3574: Marking up resources|2021-12-18|2024-05-31||[@gleachkr](https://github.com/gleachkr)||
|[3572](https://github.com/matrix-org/matrix-spec-proposals/pull/3572)|MSC3572: Relation aggregation cleanup|2021-12-16|2024-07-23||[@bwindels](https://github.com/bwindels)||
|[3571](https://github.com/matrix-org/matrix-spec-proposals/pull/3571)|MSC3571: Relation aggregation pagination|2021-12-16|2024-06-25||[@bwindels](https://github.com/bwindels)||
|[3570](https://github.com/matrix-org/matrix-spec-proposals/pull/3570)|MSC3570: Room history visibility changes for relations|2021-12-15|2022-05-05||[@bwindels](https://github.com/bwindels)||
|[3554](https://github.com/matrix-org/matrix-spec-proposals/pull/3554)|MSC3554: Extensible Events - Translatable Text|2021-12-07|2024-09-06||[@turt2live](https://github.com/turt2live)||
|[3553](https://github.com/matrix-org/matrix-spec-proposals/pull/3553)|MSC3553: Extensible Events - Videos|2021-12-07|2023-08-03||[@turt2live](https://github.com/turt2live)||
|[3552](https://github.com/matrix-org/matrix-spec-proposals/pull/3552)|MSC3552: Extensible Events - Images and Stickers|2021-12-07|2023-12-01||[@turt2live](https://github.com/turt2live)||
|[3551](https://github.com/matrix-org/matrix-spec-proposals/pull/3551)|MSC3551: Extensible Events - Files|2021-12-07|2023-04-18||[@turt2live](https://github.com/turt2live)||
|[3548](https://github.com/matrix-org/matrix-spec-proposals/pull/3548)|MSC3547: Allow appservice bot user to read any rooms the appservice is part of|2021-12-06|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[3531](https://github.com/matrix-org/matrix-spec-proposals/pull/3531)|MSC3531: Letting moderators hide messages pending moderation|2021-11-26|2023-09-23||[@Yoric](https://github.com/Yoric)||
|[3489](https://github.com/matrix-org/matrix-spec-proposals/pull/3489)|MSC3489: Sharing streams of location data with history|2021-11-14|2024-06-07||[@ara4n](https://github.com/ara4n)||
|[3488](https://github.com/matrix-org/matrix-spec-proposals/pull/3488)|MSC3488: Extending events with location data|2021-11-14|2024-04-28||[@ara4n](https://github.com/ara4n)||
|[3480](https://github.com/matrix-org/matrix-spec-proposals/pull/3480)|MSC3480: Make device names private|2021-11-09|2023-01-02||[@uhoreg](https://github.com/uhoreg)||
|[3469](https://github.com/matrix-org/matrix-spec-proposals/pull/3469)|MSC3469: Mandate HTTP Range on Content Repository Endpoints|2021-11-03|2024-04-24||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3468](https://github.com/matrix-org/matrix-spec-proposals/pull/3468)|MSC3468: MXC to Hashes|2021-11-03|2023-12-16||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3417](https://github.com/matrix-org/matrix-spec-proposals/pull/3417)|MSC3417: Call room room type|2021-09-30|2023-07-06||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3401](https://github.com/matrix-org/matrix-spec-proposals/pull/3401)|MSC3401: Native Group VoIP Signalling|2021-09-19|2024-02-12||[@ara4n](https://github.com/ara4n)||
|[3395](https://github.com/matrix-org/matrix-spec-proposals/pull/3395)|MSC3395: Synthetic appservice events|2021-09-14|2021-12-14||[@Half-Shot](https://github.com/Half-Shot)||
|[3394](https://github.com/matrix-org/matrix-spec-proposals/pull/3394)|MSC3394: New auth rule that only allows someone to post a message in relation to another message|2021-09-12|2022-05-05||[@frandavid100](https://github.com/frandavid100)||
|[3391](https://github.com/matrix-org/matrix-spec-proposals/pull/3391)|MSC3391: API to delete account data|2021-09-09|2023-11-27||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3389](https://github.com/matrix-org/matrix-spec-proposals/pull/3389)|MSC3389: Relation redactions|2021-09-09|2024-06-24||[@bwindels](https://github.com/bwindels)||
|[3386](https://github.com/matrix-org/matrix-spec-proposals/pull/3386)|MSC3386: Unified Join Rules|2021-09-07|2024-01-28||[@kevincox](https://github.com/kevincox)||
|[3382](https://github.com/matrix-org/matrix-spec-proposals/pull/3382)|MSC3382: Inline message Attachments|2021-09-07|2022-11-28||[@MurzNN](https://github.com/MurzNN)||
|[3368](https://github.com/matrix-org/matrix-spec-proposals/pull/3368)|MSC3368: Message Content Tags|2021-08-28|2021-11-26||[@0x1a8510f2](https://github.com/0x1a8510f2)||
|[3360](https://github.com/matrix-org/matrix-spec-proposals/pull/3360)|MSC3360: Server Status|2021-08-25|2022-05-05||[@daenney](https://github.com/daenney)||
|[3356](https://github.com/matrix-org/matrix-spec-proposals/pull/3356)|MSC3356: Add additional OpenID user info fields|2021-08-24|2022-11-17||[@jkanefendt](https://github.com/jkanefendt)||
|[3338](https://github.com/matrix-org/matrix-spec-proposals/pull/3338)|MSC3338: Adding iframe specifics to preview json|2021-08-13|2022-05-05||[@srdjan-catalyst](https://github.com/srdjan-catalyst)||
|[3325](https://github.com/matrix-org/matrix-spec-proposals/pull/3325)|MSC3325: Upgrading invite-only rooms|2021-08-10|2021-08-30||[@uhoreg](https://github.com/uhoreg)||
|[3309](https://github.com/matrix-org/matrix-spec-proposals/pull/3309)|MSC3309: Simple counters|2021-08-03|2022-07-15||[@erikjohnston](https://github.com/erikjohnston)||
|[3277](https://github.com/matrix-org/matrix-spec-proposals/pull/3277)|WIP: MSC3277: Scheduled messages|2021-07-10|2024-05-14||[@ara4n](https://github.com/ara4n)||
|[3266](https://github.com/matrix-org/matrix-spec-proposals/pull/3266)|MSC3266: Room summary API|2021-07-04|2024-07-26||[@deepbluev7](https://github.com/deepbluev7)||
|[3255](https://github.com/matrix-org/matrix-spec-proposals/pull/3255)|MSC3255: Use SRV record for homeservers discovery by clients|2021-06-21|2022-07-31||[@Berbe](https://github.com/Berbe)||
|[3246](https://github.com/matrix-org/matrix-spec-proposals/pull/3246)|MSC3246: Audio waveform for extensible events|2021-06-15|2022-11-14||[@turt2live](https://github.com/turt2live)||
|[3245](https://github.com/matrix-org/matrix-spec-proposals/pull/3245)|MSC3245: Voice messages (using extensible events)|2021-06-15|2023-10-02||[@turt2live](https://github.com/turt2live)||
|[3244](https://github.com/matrix-org/matrix-spec-proposals/pull/3244)|MSC3244: Room version capabilities|2021-06-15|2022-07-12||[@BillCarsonFr](https://github.com/BillCarsonFr)||
|[3230](https://github.com/matrix-org/matrix-spec-proposals/pull/3230)|MSC3230: Spaces top level order|2021-06-03|2024-05-17||[@BillCarsonFr](https://github.com/BillCarsonFr)||
|[3217](https://github.com/matrix-org/matrix-spec-proposals/pull/3217)|MSC3217: Soft kicks|2021-05-25|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[3216](https://github.com/matrix-org/matrix-spec-proposals/pull/3216)|MSC3216: Synchronized access control for Spaces|2021-05-24|2022-05-05||[@joepie91](https://github.com/joepie91)||
|[3214](https://github.com/matrix-org/matrix-spec-proposals/pull/3214)|MSC3214: Allow overriding `m.room.power_levels` using `initial_state`|2021-05-24|2024-07-05||[@tulir](https://github.com/tulir)||
|[3202](https://github.com/matrix-org/matrix-spec-proposals/pull/3202)|MSC3202: Encrypted appservices|2021-05-18|2023-07-26||[@turt2live](https://github.com/turt2live)||
|[3174](https://github.com/matrix-org/matrix-spec-proposals/pull/3174)|MSC3174: An error code for spam rejections|2021-05-04|2022-05-05||[@Yoric](https://github.com/Yoric)||
|[3160](https://github.com/matrix-org/matrix-spec-proposals/pull/3160)|MSC3160: Message timezone markup|2021-04-30|2022-05-05||[@bwindels](https://github.com/bwindels)||
|[3144](https://github.com/matrix-org/matrix-spec-proposals/pull/3144)|MSC3144: Allow Widgets By Default in Private Rooms|2021-04-23|2022-05-05||[@dbkr](https://github.com/dbkr)||
|[3131](https://github.com/matrix-org/matrix-spec-proposals/pull/3131)|MSC3131: Verifying with QR codes v2|2021-04-16|2022-05-05||[@uhoreg](https://github.com/uhoreg)||
|[3124](https://github.com/matrix-org/matrix-spec-proposals/pull/3124)|MSC3124: Handling spoilers in plain-text message fallback|2021-04-15|2022-05-05||[@xenofem](https://github.com/xenofem)||
|[3105](https://github.com/matrix-org/matrix-spec-proposals/pull/3105)|MSC3105: Previewing UIA flows|2021-04-06|2023-07-26||[@turt2live](https://github.com/turt2live)||
|[3086](https://github.com/matrix-org/matrix-spec-proposals/pull/3086)|MSC3086: Asserted identity on VoIP calls|2021-04-01|2022-05-05||[@dbkr](https://github.com/dbkr)||
|[3079](https://github.com/matrix-org/matrix-spec-proposals/pull/3079)|MSC3079: Low Bandwidth CS API|2021-03-30|2024-03-22||[@kegsay](https://github.com/kegsay)||
|[3060](https://github.com/matrix-org/matrix-spec-proposals/pull/3060)|MSC3060: Room labels|2021-03-12|2023-06-13||[@babolivier](https://github.com/babolivier)||
|[3051](https://github.com/matrix-org/matrix-spec-proposals/pull/3051)|MSC3051: A scalable relation format|2021-03-05|2023-06-06||[@deepbluev7](https://github.com/deepbluev7)||
|[3038](https://github.com/matrix-org/matrix-spec-proposals/pull/3038)|MSC3038: Typed typing notifications|2021-03-02|2022-07-11||[@turt2live](https://github.com/turt2live)||
|[3026](https://github.com/matrix-org/matrix-spec-proposals/pull/3026)|MSC3026: "busy" presence state|2021-02-23|2023-08-16||[@babolivier](https://github.com/babolivier)||
|[3015](https://github.com/matrix-org/matrix-spec-proposals/pull/3015)|MSC3015: Room state personal overrides|2021-02-19|2024-03-31||[@MurzNN](https://github.com/MurzNN)||
|[3014](https://github.com/matrix-org/matrix-spec-proposals/pull/3014)|MSC3014: HTTP Pushers for the full event with extra rooms information|2021-02-18|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[3013](https://github.com/matrix-org/matrix-spec-proposals/pull/3013)|MSC3013: Encrypted Push|2021-02-17|2024-08-01||[@Sorunome](https://github.com/Sorunome)||
|[3012](https://github.com/matrix-org/matrix-spec-proposals/pull/3012)|MSC3012: Post-registration terms of service API|2021-02-17|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[3009](https://github.com/matrix-org/matrix-spec-proposals/pull/3009)|MSC3009: Websocket transport for client <--> widget communications|2021-02-15|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[3008](https://github.com/matrix-org/matrix-spec-proposals/pull/3008)|MSC3008: Scoped access for widgets|2021-02-15|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2997](https://github.com/matrix-org/matrix-spec-proposals/pull/2997)|MSC2997: Add t-shirt|2021-02-07|2023-09-24||[@Sorunome](https://github.com/Sorunome)||
|[2974](https://github.com/matrix-org/matrix-spec-proposals/pull/2974)|MSC2974: Widgets: Capabilities re-exchange|2021-01-19|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2970](https://github.com/matrix-org/matrix-spec-proposals/pull/2970)|MSC2970: Remove pusher path requirement|2021-01-17|2023-08-21||[@Sorunome](https://github.com/Sorunome)||
|[2966](https://github.com/matrix-org/matrix-spec-proposals/pull/2966)|MSC2966: Usage of OAuth 2.0 Dynamic Client Registration in Matrix|2021-01-14|2024-09-13||[@sandhose](https://github.com/sandhose)||
|[2965](https://github.com/matrix-org/matrix-spec-proposals/pull/2965)|MSC2965: OIDC Provider discovery|2021-01-14|2024-09-04||[@sandhose](https://github.com/sandhose)||
|[2962](https://github.com/matrix-org/matrix-spec-proposals/pull/2962)|MSC2962: Managing power levels via Spaces|2021-01-14|2022-05-05||[@ara4n](https://github.com/ara4n)||
|[2961](https://github.com/matrix-org/matrix-spec-proposals/pull/2961)|MSC2961: External Signatures|2021-01-13|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2949](https://github.com/matrix-org/matrix-spec-proposals/pull/2949)|MSC2949: Proposal to clarify "Requires auth" and "Rate-limited" in the spec|2021-01-07|2024-06-01||[@tulir](https://github.com/tulir)||
|[2943](https://github.com/matrix-org/matrix-spec-proposals/pull/2943)|MSC2943: Return an event ID for membership endpoints|2021-01-07|2022-07-19||[@turt2live](https://github.com/turt2live)||
|[2938](https://github.com/matrix-org/matrix-spec-proposals/pull/2938)|MSC2938: Report content to moderators|2021-01-05|2024-04-30||[@Yoric](https://github.com/Yoric)||
|[2931](https://github.com/matrix-org/matrix-spec-proposals/pull/2931)|MSC2931: Allow widgets to navigate with matrix.to URIs|2020-12-29|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2895](https://github.com/matrix-org/matrix-spec-proposals/pull/2895)|MSC2895: Proposal to improve /members and /joined_rooms|2020-12-07|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2881](https://github.com/matrix-org/matrix-spec-proposals/pull/2881)|MSC2881: Message Attachments|2020-11-27|2022-10-13||[@MurzNN](https://github.com/MurzNN)||
|[2873](https://github.com/matrix-org/matrix-spec-proposals/pull/2873)|MSC2873: Identifying clients and user settings in widgets|2020-11-23|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2872](https://github.com/matrix-org/matrix-spec-proposals/pull/2872)|MSC2872: Move the widget title to the top level of the definition|2020-11-23|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2871](https://github.com/matrix-org/matrix-spec-proposals/pull/2871)|MSC2871: Give widgets an indication of which capabilities they were approved for|2020-11-20|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2855](https://github.com/matrix-org/matrix-spec-proposals/pull/2855)|MSC2855: Server-Initiated Client Clear-Cache & Reload|2020-11-08|2022-05-05||[@jevolk](https://github.com/jevolk)||
|[2848](https://github.com/matrix-org/matrix-spec-proposals/pull/2848)|MSC2848: Globally unique event IDs|2020-11-04|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2846](https://github.com/matrix-org/matrix-spec-proposals/pull/2846)|MSC2846: Decentralizing media through CIDs|2020-11-02|2023-06-19||[@jcgruenhage](https://github.com/jcgruenhage)||
|[2845](https://github.com/matrix-org/matrix-spec-proposals/pull/2845)|MSC2845: Phone number lookups using third party API|2020-11-02|2022-05-05||[@dbkr](https://github.com/dbkr)||
|[2839](https://github.com/matrix-org/matrix-spec-proposals/pull/2839)|MSC2839: Dynamic User-Interactive Authentication|2020-10-29|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2836](https://github.com/matrix-org/matrix-spec-proposals/pull/2836)|MSC2836: Twitter-style Threading|2020-10-28|2024-02-24||[@kegsay](https://github.com/kegsay)||
|[2835](https://github.com/matrix-org/matrix-spec-proposals/pull/2835)|MSC2835: Add UIA to the /login endpoint|2020-10-28|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2828](https://github.com/matrix-org/matrix-spec-proposals/pull/2828)|MSC2828: Proposal to restrict allowed user IDs over federation|2020-10-20|2022-05-05||[@tulir](https://github.com/tulir)||
|[2815](https://github.com/matrix-org/matrix-spec-proposals/pull/2815)|MSC2815: Proposal to allow room moderators to view redacted event content|2020-10-10|2024-04-27||[@tulir](https://github.com/tulir)||
|[2813](https://github.com/matrix-org/matrix-spec-proposals/pull/2813)|MSC2813: Widget API error handling and validation|2020-10-09|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2790](https://github.com/matrix-org/matrix-spec-proposals/pull/2790)|MSC2790: Modal widgets (acquiring user input from a widget)|2020-09-24|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2785](https://github.com/matrix-org/matrix-spec-proposals/pull/2785)|MSC2785: Event notification attributes and actions|2020-09-20|2022-05-05||[@richvdh](https://github.com/richvdh)||
|[2782](https://github.com/matrix-org/matrix-spec-proposals/pull/2782)|MSC2782: Pushers with the full event content|2020-09-19|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2772](https://github.com/matrix-org/matrix-spec-proposals/pull/2772)|MSC2772: Notifications for Jitsi Calls|2020-09-11|2022-05-05||[@dbkr](https://github.com/dbkr)||
|[2762](https://github.com/matrix-org/matrix-spec-proposals/pull/2762)|MSC2762: Allowing widgets to send/receive events|2020-09-02|2023-02-13||[@turt2live](https://github.com/turt2live)||
|[2757](https://github.com/matrix-org/matrix-spec-proposals/pull/2757)|MSC2757: Sign Events|2020-09-01|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2755](https://github.com/matrix-org/matrix-spec-proposals/pull/2755)|MSC2755: Lazy load rooms|2020-08-31|2022-05-05||[@deepbluev7](https://github.com/deepbluev7)||
|[2753](https://github.com/matrix-org/matrix-spec-proposals/pull/2753)|MSC2753: Peeking via sync (take 2)|2020-08-29|2024-06-02||[@ara4n](https://github.com/ara4n)||
|[2749](https://github.com/matrix-org/matrix-spec-proposals/pull/2749)|MSC2749: Per-user E2EE on/off setting|2020-08-25|2023-10-13||[@KB1RD](https://github.com/KB1RD)||
|[2747](https://github.com/matrix-org/matrix-spec-proposals/pull/2747)|MSC2747: VoIP call transfers|2020-08-21|2023-02-06||[@dbkr](https://github.com/dbkr)||
|[2730](https://github.com/matrix-org/matrix-spec-proposals/pull/2730)|MSC2730: Verifiable forwarded events|2020-08-12|2022-05-05||[@tulir](https://github.com/tulir)||
|[2723](https://github.com/matrix-org/matrix-spec-proposals/pull/2723)|MSC2723: Forwarded message metadata|2020-08-12|2022-12-22||[@hummlbach](https://github.com/hummlbach)||
|[2716](https://github.com/matrix-org/matrix-spec-proposals/pull/2716)|MSC2716: Incrementally importing history into existing rooms|2020-08-04|2024-06-24||[@ara4n](https://github.com/ara4n)||
|[2695](https://github.com/matrix-org/matrix-spec-proposals/pull/2695)|MSC2695: Get event by ID over federation|2020-07-23|2024-08-12||[@jryans](https://github.com/jryans)||
|[2673](https://github.com/matrix-org/matrix-spec-proposals/pull/2673)|MSC2673: Notification Levels|2020-07-07|2022-05-05||[@timokoesters](https://github.com/timokoesters)||
|[2666](https://github.com/matrix-org/matrix-spec-proposals/pull/2666)|MSC2666: Get rooms in common with another user|2020-07-05|2023-07-13||[@Half-Shot](https://github.com/Half-Shot), [@ShadowJonathan](https://github.com/ShadowJonathan)||
|[2654](https://github.com/matrix-org/matrix-spec-proposals/pull/2654)|MSC2654: Unread counts|2020-06-24|2024-08-08||[@babolivier](https://github.com/babolivier)||
|[2644](https://github.com/matrix-org/matrix-spec-proposals/pull/2644)|MSC2644: `matrix.to` URI syntax v2|2020-06-19|2024-08-12||[@jryans](https://github.com/jryans)||
|[2618](https://github.com/matrix-org/matrix-spec-proposals/pull/2618)|MSC2618: Helping others with mandatory implementation guides|2020-06-09|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2596](https://github.com/matrix-org/matrix-spec-proposals/pull/2596)|MSC2596: Proposal to always allow rescinding invites|2020-06-01|2024-04-09||[@tulir](https://github.com/tulir)||
|[2545](https://github.com/matrix-org/matrix-spec-proposals/pull/2545)|MSC2545: Image Packs (Emoticons & Stickers)|2020-05-15|2024-08-29||[@Sorunome](https://github.com/Sorunome)||
|[2529](https://github.com/matrix-org/matrix-spec-proposals/pull/2529)|MSC2529: Proposal to use existing events as captions for images|2020-05-07|2022-07-17||[@benparsons](https://github.com/benparsons)||
|[2513](https://github.com/matrix-org/matrix-spec-proposals/pull/2513)|MSC2513: Allow clients to specify content for membership events|2020-04-23|2022-05-05||[@jevolk](https://github.com/jevolk)||
|[2499](https://github.com/matrix-org/matrix-spec-proposals/pull/2499)|MSC2499: Fixes for Well-known URIs|2020-04-13|2023-03-26||[@aaronraimist](https://github.com/aaronraimist)||
|[2487](https://github.com/matrix-org/matrix-spec-proposals/pull/2487)|MSC2487: Filtering for Appservices|2020-04-03|2023-01-04||[@Sorunome](https://github.com/Sorunome)||
|[2477](https://github.com/matrix-org/matrix-spec-proposals/pull/2477)|MSC2477: User-defined ephemeral events in rooms|2020-03-28|2024-07-23||[@ananace](https://github.com/ananace)||
|[2474](https://github.com/matrix-org/matrix-spec-proposals/pull/2474)|MSC2474: Add key backup version to SSSS account data|2020-03-26|2022-05-05||[@uhoreg](https://github.com/uhoreg)||
|[2463](https://github.com/matrix-org/matrix-spec-proposals/pull/2463)|MSC2463: Exclusion of MXIDs in push rules content matching|2020-03-18|2023-05-13||[@pacien](https://github.com/pacien)||
|[2448](https://github.com/matrix-org/matrix-spec-proposals/pull/2448)|MSC2448: Using BlurHash as a Placeholder for Matrix Media|2020-02-27|2024-01-22||[@anoadragon453](https://github.com/anoadragon453)||
|[2444](https://github.com/matrix-org/matrix-spec-proposals/pull/2444)|MSC2444: peeking over federation via /peek|2020-02-24|2022-05-05||[@ara4n](https://github.com/ara4n)||
|[2437](https://github.com/matrix-org/matrix-spec-proposals/pull/2437)|MSC2437: Store tagged events in Room Account Data|2020-02-18|2023-01-20||[@giomfo](https://github.com/giomfo)||
|[2427](https://github.com/matrix-org/matrix-spec-proposals/pull/2427)|MSC2427: Proposal for JSON-based message formatting|2020-01-24|2024-01-18||[@tulir](https://github.com/tulir)||
|[2425](https://github.com/matrix-org/matrix-spec-proposals/pull/2425)|MSC2425: Remove Authentication on /submitToken Identity Service API|2020-01-24|2023-02-28||[@anoadragon453](https://github.com/anoadragon453)||
|[2416](https://github.com/matrix-org/matrix-spec-proposals/pull/2416)|MSC2416: Add m.login.jwt authentication type|2020-01-18|2022-05-24||[@Sorunome](https://github.com/Sorunome)||
|[2413](https://github.com/matrix-org/matrix-spec-proposals/pull/2413)|MSC2413: Remove client_secret|2020-01-17|2022-05-05||[@anoadragon453](https://github.com/anoadragon453)||
|[2409](https://github.com/matrix-org/matrix-spec-proposals/pull/2409)|MSC2409: Proposal to send EDUs to appservices|2020-01-14|2024-04-23||[@Sorunome](https://github.com/Sorunome)||
|[2398](https://github.com/matrix-org/matrix-spec-proposals/pull/2398)|MSC2398: proposal to allow mxc:// in the "a" tag within messages|2020-01-02|2024-03-26||[@eras](https://github.com/eras)||
|[2391](https://github.com/matrix-org/matrix-spec-proposals/pull/2391)|MSC2391: Federation point-queries.|2019-12-19|2022-05-05||[@jevolk](https://github.com/jevolk)||
|[2388](https://github.com/matrix-org/matrix-spec-proposals/pull/2388)|MSC2388: Toward the EDU-to-PDU transition: Read Receipts.|2019-12-18|2022-05-05||[@jevolk](https://github.com/jevolk)||
|[2385](https://github.com/matrix-org/matrix-spec-proposals/pull/2385)|MSC2385: Disable URL Previews, alternative method|2019-12-08|2024-01-31||[@Sorunome](https://github.com/Sorunome)|[@anoadragon453](https://github.com/anoadragon453)|
|[2380](https://github.com/matrix-org/matrix-spec-proposals/pull/2380)|MSC2380: Matrix Media Information API|2019-12-05|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2379](https://github.com/matrix-org/matrix-spec-proposals/pull/2379)|MSC2379: Add /versions endpoint to Appservice API|2019-12-04|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[2375](https://github.com/matrix-org/matrix-spec-proposals/pull/2375)|MSC2375: Appservice Invite States|2019-12-03|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2354](https://github.com/matrix-org/matrix-spec-proposals/pull/2354)|MSC2354: Device to device streaming file transfers|2019-11-14|2022-08-17||[@mvgorcum](https://github.com/mvgorcum)||
|[2346](https://github.com/matrix-org/matrix-spec-proposals/pull/2346)|MSC2346: Bridge information state event|2019-11-05|2023-02-25||[@Half-Shot](https://github.com/Half-Shot)||
|[2326](https://github.com/matrix-org/matrix-spec-proposals/pull/2326)|MSC2326: Label based filtering|2019-10-22|2023-03-29||[@ara4n](https://github.com/ara4n)||
|[2316](https://github.com/matrix-org/matrix-spec-proposals/pull/2316)|MSC2316: Federation queries to aid with database recovery|2019-10-08|2022-05-05||[@jevolk](https://github.com/jevolk)||
|[2315](https://github.com/matrix-org/matrix-spec-proposals/pull/2315)|MSC2315: Allow users to select 'none' as an integration manager|2019-10-08|2022-11-03||[@turt2live](https://github.com/turt2live)||
|[2299](https://github.com/matrix-org/matrix-spec-proposals/pull/2299)|MSC2299: Proposal to add m.textfile msgtype|2019-09-21|2022-05-05||[@Sorunome](https://github.com/Sorunome)||
|[2291](https://github.com/matrix-org/matrix-spec-proposals/pull/2291)|MSC2291: Configuration to Control Crawling|2019-09-14|2023-10-20||[@uhoreg](https://github.com/uhoreg)||
|[2278](https://github.com/matrix-org/matrix-spec-proposals/pull/2278)|MSC2278: Deleting attachments for expired and redacted messages|2019-09-03|2024-09-03||[@ara4n](https://github.com/ara4n)||
|[2270](https://github.com/matrix-org/matrix-spec-proposals/pull/2270)|MSC2270: Proposal for Ignoring Invites|2019-08-31|2022-05-05||[@ara4n](https://github.com/ara4n)||
|[2228](https://github.com/matrix-org/matrix-spec-proposals/pull/2228)|MSC2228: Self destructing events|2019-08-11|2023-04-26||[@ara4n](https://github.com/ara4n)||
|[2214](https://github.com/matrix-org/matrix-spec-proposals/pull/2214)|MSC2214: Joining upgraded private rooms|2019-08-02|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2213](https://github.com/matrix-org/matrix-spec-proposals/pull/2213)|MSC2213: Rejoinability of private/invite-only rooms|2019-08-02|2023-05-16||[@turt2live](https://github.com/turt2live)||
|[2212](https://github.com/matrix-org/matrix-spec-proposals/pull/2212)|MSC2212: Third party user power levels|2019-08-01|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2211](https://github.com/matrix-org/matrix-spec-proposals/pull/2211)|MSC2211: Identity Servers Storing Threepid Hashes at Rest|2019-08-01|2022-05-05||[@anoadragon453](https://github.com/anoadragon453)||
|[2199](https://github.com/matrix-org/matrix-spec-proposals/pull/2199)|MSC2199: Canonical DMs|2019-07-30|2024-05-25||[@turt2live](https://github.com/turt2live)||
|[2192](https://github.com/matrix-org/matrix-spec-proposals/pull/2192)|MSC2192: Inline widgets|2019-07-28|2023-04-18||[@turt2live](https://github.com/turt2live)||
|[2162](https://github.com/matrix-org/matrix-spec-proposals/pull/2162)|MSC2162: Signaling Errors at Bridges|2019-07-09|2022-06-17||[@V02460](https://github.com/V02460)||
|[2127](https://github.com/matrix-org/matrix-spec-proposals/pull/2127)|MSC2127: Federation capabilities API|2019-06-12|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[2108](https://github.com/matrix-org/matrix-spec-proposals/pull/2108)|MSC2108: Sync over Server Sent Events|2019-06-11|2022-08-16||[@stalniy](https://github.com/stalniy)||
|[2102](https://github.com/matrix-org/matrix-spec-proposals/pull/2102)|MSC2102: Enforce Canonical JSON on the wire for the S2S API|2019-06-08|2022-05-05||[@llebout](https://github.com/llebout)||
|[2063](https://github.com/matrix-org/matrix-spec-proposals/pull/2063)|MSC2063: Add "server information" public API proposal|2019-05-31|2022-05-05||[@grinapo](https://github.com/grinapo)||
|[1998](https://github.com/matrix-org/matrix-spec-proposals/pull/1998)|MSC1998: Two-Factor Authentication Providers|2019-05-14|2024-07-09||[@cyphar](https://github.com/cyphar)||
|[1974](https://github.com/matrix-org/matrix-spec-proposals/pull/1974)|MSC1974: Crypto Puzzle Challenge|2019-04-24|2023-01-18||[@Zolmeister](https://github.com/Zolmeister)||
|[1973](https://github.com/matrix-org/matrix-spec-proposals/pull/1973)|MSC1973: Hash Key User ID|2019-04-24|2022-05-05||[@Zolmeister](https://github.com/Zolmeister)||
|[1959](https://github.com/matrix-org/matrix-spec-proposals/pull/1959)|MSC1959: Sticker picker API|2019-04-09|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[1921](https://github.com/matrix-org/matrix-spec-proposals/pull/1921)|MSC1921: Support cancelling 3pid validation sessions|2019-03-08|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[1862](https://github.com/matrix-org/matrix-spec-proposals/pull/1862)|MSC1862: Presence Capabilities|2019-02-07|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[1797](https://github.com/matrix-org/matrix-spec-proposals/pull/1797)|MSC1797: Proposal for more granular profile error codes|2019-01-11|2022-05-05||[@turt2live](https://github.com/turt2live)||
|[1796](https://github.com/matrix-org/matrix-spec-proposals/pull/1796)|MSC1796: improved e2e notifications|2019-01-11|2023-03-19||[@ara4n](https://github.com/ara4n)||
|[1763](https://github.com/matrix-org/matrix-spec-proposals/pull/1763)|MSC1763: Proposal for specifying configurable message retention periods|2018-12-30|2024-05-01||[@ara4n](https://github.com/ara4n)||
|[1740](https://github.com/matrix-org/matrix-spec-proposals/pull/1740)|MSC1740: Using the Accept header to select an encoding|2018-12-02|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[1716](https://github.com/matrix-org/matrix-spec-proposals/pull/1716)|MSC1716: Open on device API|2018-11-12|2022-05-05||[@Half-Shot](https://github.com/Half-Shot)||
|[1597](https://github.com/matrix-org/matrix-spec-proposals/pull/1597)|MSC1597: Better spec for matrix identifiers|2018-08-29|2024-04-26||[@richvdh](https://github.com/richvdh)||
|[3812](https://github.com/matrix-org/matrix-spec-proposals/issues/3812)|MSC1485: Hint buttons in messages|2018-08-04|2022-05-09|[1](https://docs.google.com/document/d/1806FWv2B8_vca5ggJC_YGKgz7npbOy7Mg_9LyP74_0w/edit#)|[@tulir](https://github.com/tulir)||
|[3811](https://github.com/matrix-org/matrix-spec-proposals/issues/3811)|[WIP] MSC1453: Antivirus support|2018-07-26|2022-09-17||[@ara4n](https://github.com/ara4n)||
|[3809](https://github.com/matrix-org/matrix-spec-proposals/issues/3809)|Proposal to improve /createRoom|2018-06-22|2022-05-10|[1](https://docs.google.com/document/d/1HM5ANCyrqXVGVLERzZkYI_3x2_KpYYmtWZXnFdIV66k/edit?usp=sharing)|[@turt2live](https://github.com/turt2live)||
|[1316](https://github.com/matrix-org/matrix-spec-proposals/issues/1316)|Proposal to have the appservice registration `type` be optional|2018-06-18|2021-05-03|[1](https://docs.google.com/document/d/1ng01UlGynBfktSepGHtsCArsE5ZM6mOZzZn-ULtTZEM/edit?usp=sharing)|[@turt2live](https://github.com/turt2live)||
|[3807](https://github.com/matrix-org/matrix-spec-proposals/issues/3807)|Proposal for an application service management API|2018-06-14|2022-05-10|[1](https://docs.google.com/document/d/1Y6bWdejrOiwL5UjnJ5VnOKoK6OfK6kX-pYbWT7f5czA/edit)|[@turt2live](https://github.com/turt2live)||
|[3806](https://github.com/matrix-org/matrix-spec-proposals/issues/3806)|Proposal to filter out traffic to Appservices based on filters|2018-06-14|2022-05-10|[1](https://docs.google.com/document/d/1YhjKWTjIijdM40_4xePtU6LliDJT068IE0i09Yl1w6g/edit?usp=sharing)|[@Half-Shot](https://github.com/Half-Shot)||
|[3802](https://github.com/matrix-org/matrix-spec-proposals/issues/3802)|Mitigating abuse of the event depth parameter over federation|2018-05-10|2022-05-20|[1](https://docs.google.com/document/d/16ofbjluy8ZKYL6nt7WLHG4GqSodJUWLUxHhI6xPEjr4/edit)|[@ara4n](https://github.com/ara4n)||
|[3801](https://github.com/matrix-org/matrix-spec-proposals/issues/3801)|Bridging group membership with 3rd party group sources|2018-05-10|2022-07-19|[1](https://docs.google.com/document/d/1Nyk3Jf9BF0T2jHbeOV4DltazY5a3eP2meovSnMKDsxU/edit#heading=h.aienm7wdvf4q)|[@lukebarnard1](https://github.com/lukebarnard1)||
|[3800](https://github.com/matrix-org/matrix-spec-proposals/issues/3800)|Proposal for improved bot support|2018-05-10|2022-05-09|[1](https://docs.google.com/document/d/13ec6iqTQc7gMYGtiyP6qkzsgi3APVwuoXqJFHrfLEP4/edit?usp=sharing)|[@turt2live](https://github.com/turt2live)||
|[3799](https://github.com/matrix-org/matrix-spec-proposals/issues/3799)|Threading API|2018-05-10|2022-07-31|[1](https://docs.google.com/document/d/1bLAcYBvTYp2XNvUG-DuYv4E0uWThz_Cr6PHzspq7e60/edit)|[@Half-Shot](https://github.com/Half-Shot)||
|[3798](https://github.com/matrix-org/matrix-spec-proposals/issues/3798)|Support for websockets|2018-03-06|2022-05-09|[1](https://github.com/matrix-org/matrix-doc/blob/master/drafts/websockets.rst), [2](https://docs.google.com/document/d/104ClehFBgqLQbf4s-AKX2ijr8sOAxcizfcRs_atsB0g/edit)|[@richvdh](https://github.com/richvdh), [@krombel](https://github.com/krombel)||
|[3797](https://github.com/matrix-org/matrix-spec-proposals/issues/3797)|Refine and clarify how presence works|2017-12-26|2022-05-09|[1](https://docs.google.com/document/d/1sKaM9J5oorEeReYwOBmcgED6XnX2PdCYcx0Pp0gFnqM/edit#)|[@ara4n](https://github.com/ara4n)||
|[3796](https://github.com/matrix-org/matrix-spec-proposals/issues/3796)|Auth for content repo (and enforcing GDPR erasure)|2016-08-20|2024-04-23|[1](https://docs.google.com/document/d/1ERHpmthZyspnZtE3tQzxKTkcxar6JANeyNXgz2_djhA/edit#)|[@ara4n](https://github.com/ara4n)||
|[3795](https://github.com/matrix-org/matrix-spec-proposals/issues/3795)|Extensible Profiles. (SPEC-93)|2015-01-19|2024-03-20|[1](https://docs.google.com/document/d/1jXMElbQR-5ldt_yhWuqzLFBO3-TEJWhRyWF5Y_gGSsc/edit#heading=h.h8vj3b7rllw9)|[@erikjohnston](https://github.com/erikjohnston)||
|[3794](https://github.com/matrix-org/matrix-spec-proposals/issues/3794)|Federation API for canonicalising MXIDs|2014-10-27|2022-05-16|[1](https://docs.google.com/document/d/1B7q_3ruJzeQTg-uJHe1UScxbVLzgm451c25OjpYcojI/edit#)|[@ara4n](https://github.com/ara4n)||

### Proposed Final Comment Period[](https://spec.matrix.org/proposals/#proposed-final-comment-period)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4189](https://github.com/matrix-org/matrix-spec-proposals/pull/4189)|MSC4189: Allowing guests to access uploaded media|2024-09-05|2024-09-11||[@turt2live](https://github.com/turt2live)||
|[4183](https://github.com/matrix-org/matrix-spec-proposals/pull/4183)|MSC4183: Additional Error Codes for submitToken endpoint|2024-08-28|2024-09-05||[@dbkr](https://github.com/dbkr)||
|[4170](https://github.com/matrix-org/matrix-spec-proposals/pull/4170)|MSC4170: 403 error responses for profile APIs|2024-07-08|2024-09-11||[@Johennes](https://github.com/Johennes)||
|[4148](https://github.com/matrix-org/matrix-spec-proposals/pull/4148)|MSC4148: Permitting HTTP(S) URLs for SSO IdP icons|2024-05-29|2024-07-17||[@turt2live](https://github.com/turt2live)||
|[4133](https://github.com/matrix-org/matrix-spec-proposals/pull/4133)|MSC4133: Extending User Profile API with Key:Value Pairs|2024-04-19|2024-09-14||[@tcpipuk](https://github.com/tcpipuk)||
|[3757](https://github.com/matrix-org/matrix-spec-proposals/pull/3757)|MSC3757: Restricting who can overwrite a state event|2022-03-25|2024-09-11||[@andybalaam](https://github.com/andybalaam)||
|[2781](https://github.com/matrix-org/matrix-spec-proposals/pull/2781)|MSC2781: Remove the reply fallbacks from the specification|2020-09-18|2024-06-25||[@deepbluev7](https://github.com/deepbluev7)||

### Final Comment Period[](https://spec.matrix.org/proposals/#final-comment-period)

No MSCs are currently in this state.

### Finished Final Comment Period[](https://spec.matrix.org/proposals/#finished-final-comment-period)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[2244](https://github.com/matrix-org/matrix-spec-proposals/pull/2244)|MSC2244: Mass redactions|2019-08-23|2023-12-18||[@tulir](https://github.com/tulir)||
|[1840](https://github.com/matrix-org/matrix-spec-proposals/pull/1840)|MSC1840: Typed rooms|2019-02-03|2024-07-13||[@jfrederickson](https://github.com/jfrederickson)||

### Spec PR Missing[](https://spec.matrix.org/proposals/#spec-pr-missing)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4178](https://github.com/matrix-org/matrix-spec-proposals/pull/4178)|MSC4178: Additional Error Codes for requestToken endpoint|2024-08-16|2024-09-04||[@dbkr](https://github.com/dbkr)||
|[4163](https://github.com/matrix-org/matrix-spec-proposals/pull/4163)|MSC4163: Make ACLs apply to EDUs|2024-07-06|2024-08-05||[@Kladki](https://github.com/Kladki)||
|[4138](https://github.com/matrix-org/matrix-spec-proposals/pull/4138)|MSC4138: Update allowed HTTP methods in CORS responses|2024-05-03|2024-09-09||[@turt2live](https://github.com/turt2live)||
|[3930](https://github.com/matrix-org/matrix-spec-proposals/pull/3930)|MSC3930: Polls push rules/notifications|2022-11-14|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3381](https://github.com/matrix-org/matrix-spec-proposals/pull/3381)|MSC3381: Polls (mk II)|2021-09-07|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[2870](https://github.com/matrix-org/matrix-spec-proposals/pull/2870)|MSC2870: Protect server ACLs from redaction|2020-11-20|2024-04-15||[@turt2live](https://github.com/turt2live)||
|[2702](https://github.com/matrix-org/matrix-spec-proposals/pull/2702)|MSC2702: Specifying semantics for Content-Disposition on media|2020-07-28|2024-05-29||[@turt2live](https://github.com/turt2live)||
|[2701](https://github.com/matrix-org/matrix-spec-proposals/pull/2701)|MSC2701: Clarifying `Content-Type` usage in the media repo|2020-07-28|2024-06-25||[@turt2live](https://github.com/turt2live)||
|[1961](https://github.com/matrix-org/matrix-spec-proposals/pull/1961)|MSC1961: Integration manager authentication APIs|2019-04-09|2021-04-06||[@turt2live](https://github.com/turt2live)||
|[1957](https://github.com/matrix-org/matrix-spec-proposals/pull/1957)|MSC1957: Integration manager discovery|2019-04-08|2021-04-06||[@turt2live](https://github.com/turt2live)||
|[1767](https://github.com/matrix-org/matrix-spec-proposals/pull/1767)|MSC1767: Extensible event types & fallback in Matrix (v2)|2019-01-01|2024-08-17||[@ara4n](https://github.com/ara4n)|[@turt2live](https://github.com/turt2live%20)|

### Spec PR In Review[](https://spec.matrix.org/proposals/#spec-pr-in-review)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4151](https://github.com/matrix-org/matrix-spec-proposals/pull/4151)|MSC4151: Reporting rooms (Client-Server API)|2024-06-05|2024-08-28||[@turt2live](https://github.com/turt2live)||
|[3939](https://github.com/matrix-org/matrix-spec-proposals/pull/3939)|MSC3939: Account locking|2022-11-24|2024-08-28||[@babolivier](https://github.com/babolivier)||
|[3061](https://github.com/matrix-org/matrix-spec-proposals/pull/3061)|MSC3061: Sharing room keys for past messages|2021-03-12|2023-10-04||[@uhoreg](https://github.com/uhoreg)||
|[2774](https://github.com/matrix-org/matrix-spec-proposals/pull/2774)|MSC2774: Expose the widget ID to the widget|2020-09-11|2021-04-06||[@turt2live](https://github.com/turt2live)||
|[2765](https://github.com/matrix-org/matrix-spec-proposals/pull/2765)|MSC2765: Widget avatars|2020-09-03|2021-04-06||[@turt2live](https://github.com/turt2live)||
|[1960](https://github.com/matrix-org/matrix-spec-proposals/pull/1960)|MSC1960: OpenID information exchange with widgets|2019-04-09|2023-07-19||[@turt2live](https://github.com/turt2live)||
|[3810](https://github.com/matrix-org/matrix-spec-proposals/issues/3810)|Widget API extension: Always-on-screen|2018-07-03|2022-05-09|[1](https://docs.google.com/document/d/1_HEq5skPp1Yp559hYp2FSO2ecw8mJQR8lOvgsmYOEro/edit?usp=sharing)|[@dbkr](https://github.com/dbkr)||
|[3803](https://github.com/matrix-org/matrix-spec-proposals/issues/3803)|Matrix Widget API v2|2018-05-13|2024-05-10|[1](https://docs.google.com/document/d/1uPF7XWY_dXTKVKV7jZQ2KmsI19wn9-kFRgQ1tFQP7wQ/edit)|[@rxl881](https://github.com/rxl881)||

### Merged[](https://spec.matrix.org/proposals/#merged)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4159](https://github.com/matrix-org/matrix-spec-proposals/pull/4159)|MSC4159: Remove the deprecated name attribute on HTML anchor elements|2024-06-20|2024-07-10||[@Johennes](https://github.com/Johennes)||
|[4156](https://github.com/matrix-org/matrix-spec-proposals/pull/4156)|MSC4156: Migrate server_name to via|2024-06-17|2024-08-28||[@Johennes](https://github.com/Johennes)||
|[4132](https://github.com/matrix-org/matrix-spec-proposals/pull/4132)|MSC4132: Deprecate Linking to an Event Against a Room Alias|2024-04-17|2024-06-18||[@pixlwave](https://github.com/pixlwave)||
|[4126](https://github.com/matrix-org/matrix-spec-proposals/pull/4126)|MSC4126: Deprecation of query string auth|2024-04-10|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[4115](https://github.com/matrix-org/matrix-spec-proposals/pull/4115)|MSC4115: membership information on events|2024-02-26|2024-06-18||[@richvdh](https://github.com/richvdh)||
|[4077](https://github.com/matrix-org/matrix-spec-proposals/pull/4077)|MSC4077: Improved process for handling deprecated HTML features|2023-11-14|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[4041](https://github.com/matrix-org/matrix-spec-proposals/pull/4041)|MSC4041: http header Retry-After for http code 429|2023-08-10|2024-06-18||[@ThomasHalwax](https://github.com/ThomasHalwax)||
|[4040](https://github.com/matrix-org/matrix-spec-proposals/pull/4040)|MSC4040: Update SRV service name to IANA registration|2023-08-04|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[4026](https://github.com/matrix-org/matrix-spec-proposals/pull/4026)|MSC4026: Allow `/versions` to optionally accept authentication|2023-06-06|2024-06-18||[@H-Shay](https://github.com/H-Shay)||
|[4025](https://github.com/matrix-org/matrix-spec-proposals/pull/4025)|MSC4025: Local user erasure requests|2023-06-02|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[4010](https://github.com/matrix-org/matrix-spec-proposals/pull/4010)|MSC4010: Push rules and account data|2023-05-03|2024-06-18||[@clokep](https://github.com/clokep)||
|[4009](https://github.com/matrix-org/matrix-spec-proposals/pull/4009)|MSC4009: Expanding the Matrix ID grammar to enable E.164 IDs|2023-05-03|2024-06-18||[@clokep](https://github.com/clokep)||
|[3989](https://github.com/matrix-org/matrix-spec-proposals/pull/3989)|MSC3989: Redact `origin` property on events|2023-04-04|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3987](https://github.com/matrix-org/matrix-spec-proposals/pull/3987)|MSC3987: Push actions clean-up|2023-03-29|2024-06-18||[@clokep](https://github.com/clokep)||
|[3981](https://github.com/matrix-org/matrix-spec-proposals/pull/3981)|MSC3981: `/relations` recursion|2023-03-19|2024-06-18||[@justjanne](https://github.com/justjanne)||
|[3980](https://github.com/matrix-org/matrix-spec-proposals/pull/3980)|MSC3980: Dotted Field Consistency|2023-03-15|2024-06-18||[@clokep](https://github.com/clokep)||
|[3970](https://github.com/matrix-org/matrix-spec-proposals/pull/3970)|MSC3970: Scope transaction IDs to devices|2023-02-23|2024-06-18||[@hughns](https://github.com/hughns)||
|[3967](https://github.com/matrix-org/matrix-spec-proposals/pull/3967)|MSC3967: Do not require UIA when first uploading cross signing keys|2023-02-15|2024-06-18||[@hughns](https://github.com/hughns)||
|[3966](https://github.com/matrix-org/matrix-spec-proposals/pull/3966)|MSC3966: `event_property_contains` push rule condition|2023-02-09|2024-06-18||[@clokep](https://github.com/clokep)||
|[3958](https://github.com/matrix-org/matrix-spec-proposals/pull/3958)|MSC3958: Suppress notifications from message edits|2023-01-24|2024-06-18||[@clokep](https://github.com/clokep)||
|[3952](https://github.com/matrix-org/matrix-spec-proposals/pull/3952)|MSC3952: Intentional Mentions|2023-01-06|2024-06-18||[@clokep](https://github.com/clokep)||
|[3943](https://github.com/matrix-org/matrix-spec-proposals/pull/3943)|MSC3943: Partial joins to nameless rooms should include heroes' memberships|2022-12-02|2024-06-18||[@DMRobertson](https://github.com/DMRobertson)||
|[3938](https://github.com/matrix-org/matrix-spec-proposals/pull/3938)|MSC3938: Remove keyId from `/keys` endpoints|2022-11-22|2024-06-18||[@richvdh](https://github.com/richvdh)||
|[3925](https://github.com/matrix-org/matrix-spec-proposals/pull/3925)|MSC3925: m.replace aggregation with full event|2022-11-03|2024-06-18||[@benkuly](https://github.com/benkuly)||
|[3923](https://github.com/matrix-org/matrix-spec-proposals/pull/3923)|MSC3923: Bringing Matrix into the IETF process|2022-11-02|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916)|MSC3916: Authentication for media|2022-10-23|2024-09-05||[@richvdh](https://github.com/richvdh)||
|[3905](https://github.com/matrix-org/matrix-spec-proposals/pull/3905)|MSC3905: Application services should only be interested in local users|2022-10-12|2024-06-18||[@MadLittleMods](https://github.com/MadLittleMods)||
|[3904](https://github.com/matrix-org/matrix-spec-proposals/pull/3904)|MSC3904: Room version 10 as the default room version|2022-10-09|2024-06-18||[@FSG-Cat](https://github.com/FSG-Cat)||
|[3882](https://github.com/matrix-org/matrix-spec-proposals/pull/3882)|MSC3882: Allow an existing session to sign in a new session|2022-09-05|2024-06-18||[@hughns](https://github.com/hughns)||
|[3873](https://github.com/matrix-org/matrix-spec-proposals/pull/3873)|MSC3873: event_match dotted keys|2022-08-21|2024-06-18||[@Johennes](https://github.com/Johennes)||
|[3860](https://github.com/matrix-org/matrix-spec-proposals/pull/3860)|MSC3860: Media Download Redirects|2022-08-04|2024-06-18||[@Fizzadar](https://github.com/Fizzadar)||
|[3856](https://github.com/matrix-org/matrix-spec-proposals/pull/3856)|MSC3856: Threads List API|2022-07-26|2024-06-18||[@clokep](https://github.com/clokep)||
|[3844](https://github.com/matrix-org/matrix-spec-proposals/pull/3844)|MSC3844: Remove unused policy room sharing mechanism|2022-07-11|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3828](https://github.com/matrix-org/matrix-spec-proposals/pull/3828)|MSC3828: Content Repository CORP Headers|2022-06-01|2024-06-18||[@robertlong](https://github.com/robertlong)||
|[3827](https://github.com/matrix-org/matrix-spec-proposals/pull/3827)|MSC3827: Filtering of `/publicRooms` by room type|2022-05-27|2024-06-18||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3821](https://github.com/matrix-org/matrix-spec-proposals/pull/3821)|MSC3821: Update the redaction rules, again|2022-05-20|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3820](https://github.com/matrix-org/matrix-spec-proposals/pull/3820)|MSC3820: Room version 11|2022-05-20|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3818](https://github.com/matrix-org/matrix-spec-proposals/pull/3818)|MSC3818: Copy room type on upgrade|2022-05-18|2024-06-18||[@Mikaela](https://github.com/Mikaela)||
|[3816](https://github.com/matrix-org/matrix-spec-proposals/pull/3816)|MSC3816: Clarify Thread Participation|2022-05-17|2024-06-18||[@clokep](https://github.com/clokep)||
|[3787](https://github.com/matrix-org/matrix-spec-proposals/pull/3787)|MSC3787: Allowing knocks to restricted rooms|2022-05-04|2024-06-18||[@turt2live](https://github.com/turt2live)||
|[3786](https://github.com/matrix-org/matrix-spec-proposals/pull/3786)|MSC3786: Add a default push rule to ignore `m.room.server_acl` events|2022-04-30|2024-06-18||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3783](https://github.com/matrix-org/matrix-spec-proposals/pull/3783)|MSC3783: Fixed base64 for SAS verification|2022-04-25|2024-06-18||[@uhoreg](https://github.com/uhoreg)||
|[3773](https://github.com/matrix-org/matrix-spec-proposals/pull/3773)|MSC3773: Notifications for threads|2022-04-15|2024-06-18||[@clokep](https://github.com/clokep)||
|[3771](https://github.com/matrix-org/matrix-spec-proposals/pull/3771)|MSC3771: Read receipts for threads|2022-04-15|2024-06-18||[@clokep](https://github.com/clokep)||
|[3758](https://github.com/matrix-org/matrix-spec-proposals/pull/3758)|MSC3758: Add `event_property_is` push rule condition kind|2022-03-27|2024-06-18||[@Fizzadar](https://github.com/Fizzadar)||
|[3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743)|MSC3743: Standardized error response for unknown endpoints|2022-02-23|2024-06-18||[@clokep](https://github.com/clokep)||
|[3715](https://github.com/matrix-org/matrix-spec-proposals/pull/3715)|MSC3715: Add a pagination direction parameter to `/relations`|2022-02-07|2024-06-18||[@clokep](https://github.com/clokep)||
|[3706](https://github.com/matrix-org/matrix-spec-proposals/pull/3706)|MSC3706: Extensions to `/_matrix/federation/v2/send_join/{roomId}/{eventId}` for partial state|2022-02-03|2024-06-18||[@richvdh](https://github.com/richvdh)||
|[3700](https://github.com/matrix-org/matrix-spec-proposals/pull/3700)|MSC3700: Deprecate plaintext sender key|2022-02-03|2024-06-18||[@erikjohnston](https://github.com/erikjohnston)||
|[3676](https://github.com/matrix-org/matrix-spec-proposals/pull/3676)|MSC3676: Transitioning away from reply fallbacks|2022-01-26|2022-06-08||[@ara4n](https://github.com/ara4n)||
|[3667](https://github.com/matrix-org/matrix-spec-proposals/pull/3667)|MSC3667: Enforce integer power levels|2022-01-21|2024-06-18||[@neilalexander](https://github.com/neilalexander)||
|[3666](https://github.com/matrix-org/matrix-spec-proposals/pull/3666)|MSC3666: Bundled aggregations for server side search|2022-01-21|2024-06-18||[@clokep](https://github.com/clokep)||
|[3604](https://github.com/matrix-org/matrix-spec-proposals/pull/3604)|MSC3604: Room Version 10|2021-12-29|2022-09-27||[@turt2live](https://github.com/turt2live)||
|[3589](https://github.com/matrix-org/matrix-spec-proposals/pull/3589)|MSC3589: Room version 9 as the default room version|2021-12-23|2022-03-02||[@turt2live](https://github.com/turt2live)||
|[3582](https://github.com/matrix-org/matrix-spec-proposals/pull/3582)|MSC3582: Remove m.room.message.feedback|2021-12-21|2022-05-05||[@uhoreg](https://github.com/uhoreg)||
|[3567](https://github.com/matrix-org/matrix-spec-proposals/pull/3567)|MSC3567: Allow requesting events from the start/end of the room history|2021-12-14|2023-05-17||[@clokep](https://github.com/clokep)||
|[3550](https://github.com/matrix-org/matrix-spec-proposals/pull/3550)|MSC3550: Allow HTTP 403 as a response to profile lookups|2021-12-07|2024-06-25||[@H-Shay](https://github.com/H-Shay)||
|[3442](https://github.com/matrix-org/matrix-spec-proposals/pull/3442)|MSC3442: move the `prev_content` key to `unsigned`|2021-10-14|2024-08-31||[@richvdh](https://github.com/richvdh)||
|[3440](https://github.com/matrix-org/matrix-spec-proposals/pull/3440)|MSC3440: Threading via `m.thread` relation|2021-10-13|2024-06-18||[@germain-gg](https://github.com/germain-gg)||
|[3419](https://github.com/matrix-org/matrix-spec-proposals/pull/3419)|MSC3419: Allow guests to send more event types|2021-10-01|2022-01-05||[@ara4n](https://github.com/ara4n)||
|[3383](https://github.com/matrix-org/matrix-spec-proposals/pull/3383)|MSC3383: Include destination in X-Matrix Auth Header|2021-09-07|2024-06-18||[@jcgruenhage](https://github.com/jcgruenhage)||
|[3375](https://github.com/matrix-org/matrix-spec-proposals/pull/3375)|MSC3375: Room version 9|2021-09-02|2023-05-17||[@clokep](https://github.com/clokep)||
|[3316](https://github.com/matrix-org/matrix-spec-proposals/pull/3316)|MSC3316: Add timestamp massaging to the spec|2021-08-07|2022-07-18||[@tulir](https://github.com/tulir)||
|[3291](https://github.com/matrix-org/matrix-spec-proposals/pull/3291)|MSC3291: Muting in VoIP calls|2021-07-22|2024-03-26||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3289](https://github.com/matrix-org/matrix-spec-proposals/pull/3289)|MSC3289: Room version 8|2021-07-21|2023-05-17||[@clokep](https://github.com/clokep)||
|[3288](https://github.com/matrix-org/matrix-spec-proposals/pull/3288)|MSC3288: Add room type to `/_matrix/identity/v2/store-invite` API|2021-07-20|2022-01-17||[@BillCarsonFr](https://github.com/BillCarsonFr)||
|[3283](https://github.com/matrix-org/matrix-spec-proposals/pull/3283)|MSC3283: Expose capabilities for profile actions|2021-07-13|2022-05-16||[@JonasKress](https://github.com/JonasKress)||
|[3267](https://github.com/matrix-org/matrix-spec-proposals/pull/3267)|MSC3267: Reference relations|2021-07-05|2022-10-18||[@bwindels](https://github.com/bwindels)||
|[3231](https://github.com/matrix-org/matrix-spec-proposals/pull/3231)|MSC3231: Token authenticated registration|2021-06-04|2022-03-01||[@govynnus](https://github.com/govynnus)||
|[3173](https://github.com/matrix-org/matrix-spec-proposals/pull/3173)|MSC3173: Expose stripped state events to any potential joiner|2021-05-03|2023-05-17||[@clokep](https://github.com/clokep)||
|[3122](https://github.com/matrix-org/matrix-spec-proposals/pull/3122)|MSC3122: Deprecate starting verifications without requesting first|2021-04-14|2021-05-20||[@uhoreg](https://github.com/uhoreg)||
|[3083](https://github.com/matrix-org/matrix-spec-proposals/pull/3083)|MSC3083: Restricting room membership based on membership in other rooms|2021-03-31|2023-05-17||[@clokep](https://github.com/clokep)||
|[3077](https://github.com/matrix-org/matrix-spec-proposals/pull/3077)|MSC3077: Support for multi-stream VoIP|2021-03-29|2024-03-19||[@SimonBrandner](https://github.com/SimonBrandner)||
|[3069](https://github.com/matrix-org/matrix-spec-proposals/pull/3069)|MSC3069: Allow guests to use /account/whoami|2021-03-19|2022-02-18||[@turt2live](https://github.com/turt2live)||
|[3030](https://github.com/matrix-org/matrix-spec-proposals/pull/3030)|MSC3030: Jump to date API endpoint|2021-02-25|2022-12-21||[@MadLittleMods](https://github.com/MadLittleMods)||
|[2998](https://github.com/matrix-org/matrix-spec-proposals/pull/2998)|MSC2998: Room Version 7|2021-02-08|2021-04-30||[@anoadragon453](https://github.com/anoadragon453)||
|[2946](https://github.com/matrix-org/matrix-spec-proposals/pull/2946)|MSC2946: Spaces Summary|2021-01-07|2023-08-15||[@clokep](https://github.com/clokep)||
|[2918](https://github.com/matrix-org/matrix-spec-proposals/pull/2918)|MSC2918: Refresh tokens|2020-12-18|2022-10-14||[@sandhose](https://github.com/sandhose)||
|[2874](https://github.com/matrix-org/matrix-spec-proposals/pull/2874)|MSC2874: Single SSSS|2020-11-23|2021-04-28||[@uhoreg](https://github.com/uhoreg)||
|[2867](https://github.com/matrix-org/matrix-spec-proposals/pull/2867)|MSC2867: Marking rooms as unread|2020-11-17|2024-07-15||[@Bubu](https://github.com/Bubu)||
|[2858](https://github.com/matrix-org/matrix-spec-proposals/pull/2858)|MSC2858: Multiple SSO Identity Providers|2020-11-09|2021-05-27||[@t3chguy](https://github.com/t3chguy)||
|[2844](https://github.com/matrix-org/matrix-spec-proposals/pull/2844)|MSC2844: Global version number for the whole spec|2020-10-30|2022-02-22||[@turt2live](https://github.com/turt2live)||
|[2832](https://github.com/matrix-org/matrix-spec-proposals/pull/2832)|MSC2832: HS -> AS authorization header|2020-10-24|2022-08-04||[@tulir](https://github.com/tulir)||
|[2801](https://github.com/matrix-org/matrix-spec-proposals/pull/2801)|MSC2801: Make it explicit that event bodies are untrusted data|2020-10-01|2022-05-19||[@richvdh](https://github.com/richvdh)||
|[2788](https://github.com/matrix-org/matrix-spec-proposals/pull/2788)|MSC2788: Room version 6 as the default room version|2020-09-23|2020-10-05||[@turt2live](https://github.com/turt2live)||
|[2778](https://github.com/matrix-org/matrix-spec-proposals/pull/2778)|MSC2778: Providing authentication method for appservice users|2020-09-16|2021-12-28||[@Half-Shot](https://github.com/Half-Shot)||
|[2758](https://github.com/matrix-org/matrix-spec-proposals/pull/2758)|MSC2758: Proposal for a common identifier grammar|2020-09-01|2021-12-29||[@richvdh](https://github.com/richvdh)||
|[2746](https://github.com/matrix-org/matrix-spec-proposals/pull/2746)|MSC2746: Improved VoIP Signalling|2020-08-21|2023-05-23||[@dbkr](https://github.com/dbkr)||
|[2732](https://github.com/matrix-org/matrix-spec-proposals/pull/2732)|MSC2732: Olm fallback keys|2020-08-14|2022-01-05||[@uhoreg](https://github.com/uhoreg)||
|[2713](https://github.com/matrix-org/matrix-spec-proposals/pull/2713)|MSC2713: Remove deprecated v1 Identity Service API|2020-07-30|2022-07-04||[@turt2live](https://github.com/turt2live)||
|[2705](https://github.com/matrix-org/matrix-spec-proposals/pull/2705)|MSC2705: Animated thumbnails for media|2020-07-28|2024-04-01||[@turt2live](https://github.com/turt2live)||
|[2689](https://github.com/matrix-org/matrix-spec-proposals/pull/2689)|MSC2689: Fix E2EE for guests|2020-07-15|2020-10-07||[@awesome-michael](https://github.com/awesome-michael)||
|[2677](https://github.com/matrix-org/matrix-spec-proposals/pull/2677)|MSC2677: Annotations and reactions|2020-07-07|2023-04-25||[@uhoreg](https://github.com/uhoreg)|[@richvdh](https://github.com/richvdh%20)|
|[2676](https://github.com/matrix-org/matrix-spec-proposals/pull/2676)|MSC2676: Message editing|2020-07-07|2022-11-08||[@uhoreg](https://github.com/uhoreg)||
|[2675](https://github.com/matrix-org/matrix-spec-proposals/pull/2675)|MSC2675: Serverside aggregations of message relationships|2020-07-07|2022-06-08||[@uhoreg](https://github.com/uhoreg)||
|[2674](https://github.com/matrix-org/matrix-spec-proposals/pull/2674)|MSC2674: Event Relationships|2020-07-07|2023-02-23||[@uhoreg](https://github.com/uhoreg)||
|[2663](https://github.com/matrix-org/matrix-spec-proposals/pull/2663)|MSC2663: Errors for dealing with non-existent push rules|2020-07-03|2020-10-07||[@reivilibre](https://github.com/reivilibre)||
|[2659](https://github.com/matrix-org/matrix-spec-proposals/pull/2659)|MSC2659: Application service ping endpoint|2020-06-29|2024-06-18||[@tulir](https://github.com/tulir)||
|[2630](https://github.com/matrix-org/matrix-spec-proposals/pull/2630)|MSC2630: checking public keys in SAS verification|2020-06-11|2021-05-01||[@uhoreg](https://github.com/uhoreg)||
|[2611](https://github.com/matrix-org/matrix-spec-proposals/pull/2611)|MSC2611: Remove `m.login.token` User-Interactive Authentication type from the specification|2020-06-05|2022-03-01||[@richvdh](https://github.com/richvdh)||
|[2610](https://github.com/matrix-org/matrix-spec-proposals/pull/2610)|MSC2610: Remove `m.login.oauth2` User-Interactive Authentication type from the specification|2020-06-05|2022-02-21||[@richvdh](https://github.com/richvdh)||
|[2604](https://github.com/matrix-org/matrix-spec-proposals/pull/2604)|MSC2604: Accept device information for the login fallback endpoint|2020-06-04|2023-05-17||[@clokep](https://github.com/clokep)||
|[2582](https://github.com/matrix-org/matrix-spec-proposals/pull/2582)|MSC2582: Remove mimetype from EncryptedFile object|2020-05-26|2021-09-27||[@Sorunome](https://github.com/Sorunome)||
|[2557](https://github.com/matrix-org/matrix-spec-proposals/pull/2557)|MSC2557: Proposal to clarify spoilers|2020-05-19|2021-04-06||[@turt2live](https://github.com/turt2live)||
|[2540](https://github.com/matrix-org/matrix-spec-proposals/pull/2540)|MSC2540: Stricter event validation: JSON compliance|2020-05-13|2023-05-17||[@clokep](https://github.com/clokep)||
|[2530](https://github.com/matrix-org/matrix-spec-proposals/pull/2530)|MSC2530: Body field as media caption|2020-05-07|2024-02-26||[@tulir](https://github.com/tulir)||
|[2526](https://github.com/matrix-org/matrix-spec-proposals/pull/2526)|MSC2526: Add ability to delete key backups|2020-05-05|2020-06-02||[@uhoreg](https://github.com/uhoreg)||
|[2472](https://github.com/matrix-org/matrix-spec-proposals/pull/2472)|MSC2472: Symmetric SSSS|2020-03-24|2023-04-12||[@uhoreg](https://github.com/uhoreg)||
|[2457](https://github.com/matrix-org/matrix-spec-proposals/pull/2457)|MSC2457: Invalidating devices during password modification|2020-03-12|2023-05-17||[@clokep](https://github.com/clokep)||
|[2454](https://github.com/matrix-org/matrix-spec-proposals/pull/2454)|MSC2454: Support UI auth for SSO|2020-03-09|2023-05-17||[@clokep](https://github.com/clokep)||
|[2451](https://github.com/matrix-org/matrix-spec-proposals/pull/2451)|MSC2451: Remove `query_auth` federation endpoint.|2020-03-03|2020-04-22||[@clokep](https://github.com/clokep)||
|[2432](https://github.com/matrix-org/matrix-spec-proposals/pull/2432)|MSC2432: Updated semantics for publishing room aliases|2020-02-10|2023-07-28||[@richvdh](https://github.com/richvdh)||
|[2422](https://github.com/matrix-org/matrix-spec-proposals/pull/2422)|MSC2422: Allow color on font tag|2020-01-23|2021-04-06||[@deepbluev7](https://github.com/deepbluev7)||
|[2414](https://github.com/matrix-org/matrix-spec-proposals/pull/2414)|MSC2414: Make reason and score parameters optional for reporting content|2020-01-18|2022-03-01||[@iinuwa](https://github.com/iinuwa)||
|[2403](https://github.com/matrix-org/matrix-spec-proposals/pull/2403)|MSC2403: Add "knock" feature|2020-01-09|2022-06-27||[@Sorunome](https://github.com/Sorunome)|[@anoadragon453](https://github.com/anoadragon453)|
|[2399](https://github.com/matrix-org/matrix-spec-proposals/pull/2399)|MSC2399: Reporting that decryption keys are withheld|2020-01-02|2022-07-15||[@uhoreg](https://github.com/uhoreg)||
|[2367](https://github.com/matrix-org/matrix-spec-proposals/pull/2367)|MSC2367: Add reason field to all membership events|2019-11-26|2020-10-07||[@erikjohnston](https://github.com/erikjohnston)||
|[2366](https://github.com/matrix-org/matrix-spec-proposals/pull/2366)|MSC2366: Key verification flow additions: m.key.verification.ready and m.key.verification.done|2019-11-25|2022-03-01||[@uhoreg](https://github.com/uhoreg)||
|[2334](https://github.com/matrix-org/matrix-spec-proposals/pull/2334)|MSC2334 - Change default room version to v5|2019-10-30|2020-04-20||[@aaronraimist](https://github.com/aaronraimist)||
|[2324](https://github.com/matrix-org/matrix-spec-proposals/pull/2324)|MSC2324: Facilitating early releases of software dependent on spec|2019-10-18|2020-05-21||[@turt2live](https://github.com/turt2live)||
|[2320](https://github.com/matrix-org/matrix-spec-proposals/pull/2320)|MSC2320: Versions information for identity servers|2019-10-15|2021-05-04||[@babolivier](https://github.com/babolivier)||
|[2313](https://github.com/matrix-org/matrix-spec-proposals/pull/2313)|MSC2313: Moderation policies as rooms (ban lists)|2019-10-05|2020-11-14||[@ara4n](https://github.com/ara4n)||
|[2312](https://github.com/matrix-org/matrix-spec-proposals/pull/2312)|MSC2312: Matrix URI scheme proposal|2019-10-04|2022-03-01||[@KitsuneRal](https://github.com/KitsuneRal)||
|[2290](https://github.com/matrix-org/matrix-spec-proposals/pull/2290)|MSC2290: Separate Endpoints for Threepid Binding|2019-09-12|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2285](https://github.com/matrix-org/matrix-spec-proposals/pull/2285)|MSC2285: Private read receipts|2019-09-06|2022-09-15||[@SimonBrandner](https://github.com/SimonBrandner)||
|[2284](https://github.com/matrix-org/matrix-spec-proposals/pull/2284)|MSC2284: Making the identity server optional during discovery|2019-09-06|2021-05-02||[@turt2live](https://github.com/turt2live)||
|[2265](https://github.com/matrix-org/matrix-spec-proposals/pull/2265)|MSC2265: Proposal for mandating case folding when processing e-mail address localparts|2019-08-30|2021-07-12||[@babolivier](https://github.com/babolivier)||
|[2264](https://github.com/matrix-org/matrix-spec-proposals/pull/2264)|MSC2264: Add an unstable feature flag to MSC2140 for clients to detect support|2019-08-29|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[2263](https://github.com/matrix-org/matrix-spec-proposals/pull/2263)|MSC2263: Give homeservers the ability to handle their own 3PID registrations/password resets|2019-08-29|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[2249](https://github.com/matrix-org/matrix-spec-proposals/pull/2249)|MSC2249: Require users to have visibility on an event when submitting reports|2019-08-27|2023-08-15||[@Half-Shot](https://github.com/Half-Shot)||
|[2246](https://github.com/matrix-org/matrix-spec-proposals/pull/2246)|MSC2246: Asynchronous media uploads|2019-08-24|2023-11-30||[@tulir](https://github.com/tulir)||
|[2241](https://github.com/matrix-org/matrix-spec-proposals/pull/2241)|MSC2241: Key verification in DMs|2019-08-22|2021-07-05||[@uhoreg](https://github.com/uhoreg)||
|[2240](https://github.com/matrix-org/matrix-spec-proposals/pull/2240)|MSC2240: Room version 6|2019-08-22|2020-07-29||[@turt2live](https://github.com/turt2live)||
|[2230](https://github.com/matrix-org/matrix-spec-proposals/pull/2230)|MSC2230: Store Identity Server in Account Data|2019-08-13|2020-04-20||[@dbkr](https://github.com/dbkr)||
|[2209](https://github.com/matrix-org/matrix-spec-proposals/pull/2209)|MSC2209: Alter auth rules to check notifications in m.room.power_levels|2019-08-01|2020-07-29||[@lucavb](https://github.com/lucavb)||
|[2197](https://github.com/matrix-org/matrix-spec-proposals/pull/2197)|MSC2197: Search Filtering in Federation /publicRooms|2019-07-29|2021-10-30||[@reivilibre](https://github.com/reivilibre)||
|[2191](https://github.com/matrix-org/matrix-spec-proposals/pull/2191)|MSC2191: Markup for mathematical messages|2019-07-26|2024-05-28||[@uhoreg](https://github.com/uhoreg)||
|[2184](https://github.com/matrix-org/matrix-spec-proposals/pull/2184)|MSC2184: Allow the use of the HTML <details> tag|2019-07-16|2021-04-06||[@ananace](https://github.com/ananace)||
|[2181](https://github.com/matrix-org/matrix-spec-proposals/pull/2181)|MSC2181: Add an Error Code for Signaling a Deactivated User|2019-07-16|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2176](https://github.com/matrix-org/matrix-spec-proposals/pull/2176)|MSC2176: Update the redaction rules|2019-07-14|2023-08-15||[@richvdh](https://github.com/richvdh)||
|[2175](https://github.com/matrix-org/matrix-spec-proposals/pull/2175)|MSC2175: Remove the `creator` field from `m.room.create` events|2019-07-14|2023-08-15||[@richvdh](https://github.com/richvdh)||
|[2174](https://github.com/matrix-org/matrix-spec-proposals/pull/2174)|MSC2174: Move the `redacts` key to a sane place|2019-07-14|2023-08-15||[@richvdh](https://github.com/richvdh)||
|[2140](https://github.com/matrix-org/matrix-spec-proposals/pull/2140)|MSC2140: Terms of Service for ISes and IMs|2019-06-20|2022-03-01||[@dbkr](https://github.com/dbkr)||
|[2134](https://github.com/matrix-org/matrix-spec-proposals/pull/2134)|MSC2134: Identity Hash Lookups|2019-06-15|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2078](https://github.com/matrix-org/matrix-spec-proposals/pull/2078)|MSC2078: Sending Third-Party Request Tokens via the Homeserver|2019-06-05|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2077](https://github.com/matrix-org/matrix-spec-proposals/pull/2077)|MSC2077: room v5|2019-06-04|2024-04-24||[@richvdh](https://github.com/richvdh)||
|[2076](https://github.com/matrix-org/matrix-spec-proposals/pull/2076)|MSC2076: Enforce key-validity periods when validating event signatures|2019-06-04|2024-04-24||[@richvdh](https://github.com/richvdh)||
|[2033](https://github.com/matrix-org/matrix-spec-proposals/pull/2033)|MSC2033: Adding a device_id to /account/whoami|2019-05-27|2021-05-03||[@turt2live](https://github.com/turt2live)||
|[2010](https://github.com/matrix-org/matrix-spec-proposals/pull/2010)|MSC2010: Add client-side spoilers|2019-05-22|2022-03-01||[@Sorunome](https://github.com/Sorunome)||
|[2002](https://github.com/matrix-org/matrix-spec-proposals/pull/2002)|MSC2002: Proposal for adopting MSC1884 as v4 rooms|2019-05-17|2020-04-20||[@ara4n](https://github.com/ara4n)||
|[1983](https://github.com/matrix-org/matrix-spec-proposals/pull/1983)|MSC1983: Supporting a reason for leaving rooms|2019-04-30|2021-05-01||[@turt2live](https://github.com/turt2live)||
|[1954](https://github.com/matrix-org/matrix-spec-proposals/pull/1954)|MSC1954: Proposal to remove prev_content from the essential keys list|2019-04-05|2020-04-20||[@neilisfragile](https://github.com/neilisfragile)||
|[1946](https://github.com/matrix-org/matrix-spec-proposals/pull/1946)|MSC1946: Secure Secret Storage and Sharing|2019-03-28|2023-04-12||[@uhoreg](https://github.com/uhoreg)||
|[1930](https://github.com/matrix-org/matrix-spec-proposals/pull/1930)|MSC1930: Add a push rule for m.room.tombstone events|2019-03-15|2022-08-31||[@turt2live](https://github.com/turt2live)||
|[1929](https://github.com/matrix-org/matrix-spec-proposals/pull/1929)|MSC1929: Homeserver Admin Contact and Support page|2019-03-15|2024-06-18||[@Half-Shot](https://github.com/Half-Shot)||
|[1915](https://github.com/matrix-org/matrix-spec-proposals/pull/1915)|MSC 1915 - Add a 3PID unbind API|2019-03-06|2020-04-20||[@erikjohnston](https://github.com/erikjohnston)||
|[1884](https://github.com/matrix-org/matrix-spec-proposals/pull/1884)|MSC1884: Proposal to replace slashes in event IDs|2019-02-13|2020-04-20||[@richvdh](https://github.com/richvdh)||
|[1866](https://github.com/matrix-org/matrix-spec-proposals/pull/1866)|Add proposal for invite error code for unsupported room version|2019-02-08|2020-04-20||[@erikjohnston](https://github.com/erikjohnston)||
|[1831](https://github.com/matrix-org/matrix-spec-proposals/pull/1831)|MSC1831: Change the order of .well-known and SRV discovery techniques|2019-01-31|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[1819](https://github.com/matrix-org/matrix-spec-proposals/pull/1819)|MSC:1819 Remove Presence Lists|2019-01-28|2020-04-20||[@neilisfragile](https://github.com/neilisfragile)||
|[1813](https://github.com/matrix-org/matrix-spec-proposals/pull/1813)|MSC 1813 - Federation Make Membership Room Version|2019-01-22|2020-04-20||[@erikjohnston](https://github.com/erikjohnston)||
|[1804](https://github.com/matrix-org/matrix-spec-proposals/pull/1804)|MSC1804: Advertising capable room versions to clients|2019-01-17|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[1802](https://github.com/matrix-org/matrix-spec-proposals/pull/1802)|MSC1802: Remove the '200' value from some federation responses|2019-01-14|2020-05-19||[@babolivier](https://github.com/babolivier)||
|[1794](https://github.com/matrix-org/matrix-spec-proposals/pull/1794)|MSC 1794 - Federation v2 Invite API|2019-01-10|2021-09-22||[@erikjohnston](https://github.com/erikjohnston)||
|[1779](https://github.com/matrix-org/matrix-spec-proposals/pull/1779)|MSC1779: Proposal for Open Governance for Matrix.org (v2)|2019-01-07|2022-05-06||[@ara4n](https://github.com/ara4n)||
|[1772](https://github.com/matrix-org/matrix-spec-proposals/pull/1772)|MSC1772: Matrix spaces|2019-01-03|2022-03-01||[@ara4n](https://github.com/ara4n)||
|[1759](https://github.com/matrix-org/matrix-spec-proposals/pull/1759)|Room v2 proposal|2018-12-17|2020-04-20||[@erikjohnston](https://github.com/erikjohnston)||
|[1756](https://github.com/matrix-org/matrix-spec-proposals/pull/1756)|MSC1756: cross-signing devices using a master identity key|2018-12-14|2020-12-15||[@uhoreg](https://github.com/uhoreg)||
|[1753](https://github.com/matrix-org/matrix-spec-proposals/pull/1753)|MSC1753: client-server capabilities API|2018-12-12|2020-04-20||[@richvdh](https://github.com/richvdh)||
|[1730](https://github.com/matrix-org/matrix-spec-proposals/pull/1730)|MSC1730: Mechanism for redirecting to an alternative server during login|2018-11-23|2020-04-20||[@richvdh](https://github.com/richvdh)||
|[1721](https://github.com/matrix-org/matrix-spec-proposals/pull/1721)|MSC1721: Rename m.login.cas to m.login.sso|2018-11-15|2021-09-22||[@richvdh](https://github.com/richvdh)||
|[1719](https://github.com/matrix-org/matrix-spec-proposals/pull/1719)|MSC1719: olm session unwedging|2018-11-14|2024-03-15||[@uhoreg](https://github.com/uhoreg)||
|[1717](https://github.com/matrix-org/matrix-spec-proposals/pull/1717)|MSC1717: common definitions for key verification methods|2018-11-13|2020-04-20||[@uhoreg](https://github.com/uhoreg)||
|[1711](https://github.com/matrix-org/matrix-spec-proposals/pull/1711)|MSC1711: X.509 certificate verification for federation connections|2018-11-07|2022-03-01||[@richvdh](https://github.com/richvdh)||
|[1708](https://github.com/matrix-org/matrix-spec-proposals/pull/1708)|MSC1708: .well-known support for server name resolution|2018-11-05|2022-12-17||[@richvdh](https://github.com/richvdh)||
|[1704](https://github.com/matrix-org/matrix-spec-proposals/pull/1704)|MSC1704: Adding ?via= to matrix.to permalinks to help with routing|2018-10-26|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[1693](https://github.com/matrix-org/matrix-spec-proposals/pull/1693)|MSC1693: Specify how to handle rejected events in new state res|2018-10-08|2020-04-20||[@erikjohnston](https://github.com/erikjohnston)||
|[1692](https://github.com/matrix-org/matrix-spec-proposals/pull/1692)|MSC1692: Terms of service at registration|2018-10-03|2024-05-10||[@turt2live](https://github.com/turt2live)||
|[1659](https://github.com/matrix-org/matrix-spec-proposals/pull/1659)|MSC 1659 Proposal: Change Event IDs to Hashes|2018-09-05|2020-04-21||[@erikjohnston](https://github.com/erikjohnston)||
|[1544](https://github.com/matrix-org/matrix-spec-proposals/pull/1544)|MSC1544: Key verification using QR codes|2018-08-20|2022-03-01||[@uhoreg](https://github.com/uhoreg)||
|[1504](https://github.com/matrix-org/matrix-spec-proposals/issues/1504)|Homeserver resource limiting error codes|2018-08-13|2020-04-21|[1](https://docs.google.com/document/d/1_CgwkfLznU56fwLUFZXFnKivzVUPjjNsTO0DpCB9ncM/edit?usp=sharing)|[@neilisfragile](https://github.com/neilisfragile)||
|[1501](https://github.com/matrix-org/matrix-spec-proposals/issues/1501)|Room version upgrades|2018-08-10|2020-04-21|[1](https://github.com/matrix-org/matrix-doc/blob/master/proposals/1501-room-version-upgrades.md)|[@richvdh](https://github.com/richvdh)||
|[1497](https://github.com/matrix-org/matrix-spec-proposals/issues/1497)|MSC1497: Advertising support of experimental features in the CS API|2018-08-08|2020-04-21|[1](https://docs.google.com/document/d/10t3Ehz1JZWdDCSAS3SvxZZwb3cXYnJBoOPL4tcIKNYg/edit#heading=h.h8g8a1l2xtks)|[@ara4n](https://github.com/ara4n)||
|[1466](https://github.com/matrix-org/matrix-spec-proposals/issues/1466)|MSC1466: Soft Logout|2018-08-01|2020-05-26|[1](https://github.com/matrix-org/matrix-doc/pull/1467)|[@erikjohnston](https://github.com/erikjohnston)||
|[1452](https://github.com/matrix-org/matrix-spec-proposals/issues/1452)|Homeserver Warning Messages|2018-07-25|2020-04-21|[1](https://docs.google.com/document/d/1R_a1zi5qRe06D6D3fNZaKTqQSyaKcQT2Ejt4kImt6yo/edit#)|[@dbkr](https://github.com/dbkr)||
|[1442](https://github.com/matrix-org/matrix-spec-proposals/issues/1442)|State Resolution: Reloaded|2018-07-20|2020-04-21|[1](https://github.com/matrix-org/matrix-doc/pull/1441)|[@erikjohnston](https://github.com/erikjohnston)||
|[1426](https://github.com/matrix-org/matrix-spec-proposals/issues/1426)|Proposal for clarifying and improving review process for MSCs|2018-07-17|2020-04-21|[1](https://docs.google.com/document/d/1miOde4v98lwxZNQqT40ksBcXGdliign8rfmobDfC5u0/edit)|[@ara4n](https://github.com/ara4n)||
|[1425](https://github.com/matrix-org/matrix-spec-proposals/issues/1425)|Room Versioning|2018-07-17|2020-04-21|[1](https://docs.google.com/document/d/1urKgReoHqxX8R_XtySB17dPi-DZcKhqTEL2_s895Wz0)|[@richvdh](https://github.com/richvdh)||
|[1383](https://github.com/matrix-org/matrix-spec-proposals/issues/1383)|Proposal for ACLing servers from rooms|2018-07-05|2020-04-21|[1](https://docs.google.com/document/d/1aiuROf1__7ZFkJvDdAZQfBNxyzjYd-ijiRAcHJYqJCM/edit#heading=h.t1ebd56v2ae6)|[@richvdh](https://github.com/richvdh), [@ara4n](https://github.com/ara4n)||
|[1339](https://github.com/matrix-org/matrix-spec-proposals/issues/1339)|Proposal to add a GET method to read account data|2018-06-24|2020-04-21||[@ananace](https://github.com/ananace)||
|[1323](https://github.com/matrix-org/matrix-spec-proposals/issues/1323)|AS traffic rate-limiting|2018-06-20|2018-08-28|[1](https://docs.google.com/document/d/14ygfhAMUrAa04YMHHl2P8_mxV3H2ntNq_-crmZizED0/edit?usp=sharing)|[@anoadragon453](https://github.com/anoadragon453)||
|[1304](https://github.com/matrix-org/matrix-spec-proposals/issues/1304)|Proposal to simplify the auth rules of m.room.power_level events.|2018-06-14|2020-04-21|[1](https://docs.google.com/document/d/1YuaCFH3RzBUIAjJWFMzKROMDlttoP94KIsyV_F_kfNs/edit#heading=h.8b2tmd2n0vhz)|[@richvdh](https://github.com/richvdh), [@ara4n](https://github.com/ara4n)||
|[1267](https://github.com/matrix-org/matrix-spec-proposals/issues/1267)|MSC 1267: Interactive key verification using short authentication strings|2018-05-28|2020-04-21|[1](https://docs.google.com/document/d/1SXmyjyNqClJ5bTHtwvp8tT1Db4pjlGVxfPQNdlQILqU/edit#)|[@uhoreg](https://github.com/uhoreg)||
|[1234](https://github.com/matrix-org/matrix-spec-proposals/issues/1234)|Rich Replies format|2018-05-12|2020-04-21|[1](https://docs.google.com/document/d/1BPd4lBrooZrWe_3s_lHw_e-Dydvc7bXbm02_sV2k6Sc)|[@t3chguy](https://github.com/t3chguy)||
|[1233](https://github.com/matrix-org/matrix-spec-proposals/issues/1233)|A proposal for organising spec proposals|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1wLln7da12l0H5YgAh5xM2TVE7VsTjXzhEwVh3sRBMCk/edit#)|[@ara4n](https://github.com/ara4n)||
|[1232](https://github.com/matrix-org/matrix-spec-proposals/issues/1232)|Media limits API|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1fI4ZqQcyAyBEPMtS1MCZWpN84kEPdm9SDw6SVZsJvYY/edit)|[@Half-Shot](https://github.com/Half-Shot)||
|[1230](https://github.com/matrix-org/matrix-spec-proposals/issues/1230)|Temporary mitigation for depth parameter abuse|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1I3fi2S-XnpO45qrpCsowZv8P8dHcNZ4fsBsbOW7KABI/edit#heading=h.fj95ykuss7s1)|[@ara4n](https://github.com/ara4n)||
|[1227](https://github.com/matrix-org/matrix-spec-proposals/issues/1227)|Proposal for lazy-loading room members to improve initial sync speed and client RAM usage|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/11yn-mAkYll10RJpN0mkYEVqraTbU3U4eQx9MNrzqX1U/edit)|[@ara4n](https://github.com/ara4n)||
|[1219](https://github.com/matrix-org/matrix-spec-proposals/issues/1219)|Proposal for storing megolm keys serverside|2018-05-10|2020-06-02|[1](https://github.com/matrix-org/matrix-doc/pull/1538)|[@ara4n](https://github.com/ara4n), [@uhoreg](https://github.com/uhoreg)||
|[1212](https://github.com/matrix-org/matrix-spec-proposals/issues/1212)|Device List Update Stream|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1fNBZUeMlp0fn0en5bCji5fn6mSvj48UylWfGKrk8ZIw/edit#heading=h.j3k62x61k895)|[@richvdh](https://github.com/richvdh)||
|[1211](https://github.com/matrix-org/matrix-spec-proposals/issues/1211)|Megolm session export format|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1UjWpNMfof3ynFbEOtHWGmqxy_WrFZEojrGWUq_os0G8/edit)|[@richvdh](https://github.com/richvdh)||
|[1208](https://github.com/matrix-org/matrix-spec-proposals/issues/1208)|Encrypted attachment format|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1vZi2xGmWLQMANobe5IxaqxiFc4HhykZDNcu102xjZlQ/edit)|[@NegativeMjark](https://github.com/NegativeMjark)||
|[1207](https://github.com/matrix-org/matrix-spec-proposals/issues/1207)|Publishing Room Lists for 3rd party networks|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/12mVuOT7Qoa49L_PQAPjavoK9c2nalYEFOHxJOmH5j-w/edit)|[@erikjohnston](https://github.com/erikjohnston)||
|[1205](https://github.com/matrix-org/matrix-spec-proposals/issues/1205)|Proposal for multi-device deletion API|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1LaA9Q96ytumLmE-eAscONMMX5rE26ri4G7uj-rmltbs/edit)|[@richvdh](https://github.com/richvdh)||
|[1204](https://github.com/matrix-org/matrix-spec-proposals/issues/1204)|Access Token Semantics (refresh and macaroons) - aka Auth Sept 2016 Edition|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1mdis1LQcoOSVRElszEmrAWZGIX0jX_croSha-X5oe_w/edit#heading=h.3zmkga77kwe3)|[@richvdh](https://github.com/richvdh)||
|[1203](https://github.com/matrix-org/matrix-spec-proposals/issues/1203)|3rd Party Entity lookup API|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/13NGa46a_WWno-XYfe8mQrglQdtOYMFVZtxkPKXDC3ac/edit#heading=h.m0btedxhv6ao)|[@leonerd](https://github.com/leonerd)||
|[1201](https://github.com/matrix-org/matrix-spec-proposals/issues/1201)|Device Management API|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1H8Z5b9kGKuvFkfDR1uQHaKdYxBD03ZDjMGH1IXQ0Wbw/edit#heading=h.8rtccxo23ng)|[@richvdh](https://github.com/richvdh)||
|[1200](https://github.com/matrix-org/matrix-spec-proposals/issues/1200)|Configuration of E2E encryption in a room|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1SEPMhNh6ztcrrbkGRSayVQ23bd3cfMPkTgGL4kBS9Ps/edit#heading=h.e7hfigo2zcsj)|[@richvdh](https://github.com/richvdh)||
|[1199](https://github.com/matrix-org/matrix-spec-proposals/issues/1199)|Notifications API|2018-05-10|2018-06-25|[1](https://docs.google.com/document/d/1tQUOkbygHky_6Te4ZNCju_KV0Phmk1cuJsbf2Ir0Vvs/edit)|[@dbkr](https://github.com/dbkr)||
|[1197](https://github.com/matrix-org/matrix-spec-proposals/issues/1197)|Ignoring Users|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1Jex7lDAwmv0KcgyL9oeGfUCsjw0CWSqedPKZ1ViSVis/edit)|[@erikjohnston](https://github.com/erikjohnston)||
|[1183](https://github.com/matrix-org/matrix-spec-proposals/issues/1183)|Document key-share requests|2018-04-30|2020-04-21|[1](https://docs.google.com/document/d/1m4gQkcnJkxNuBmb5NoFCIadIY-DyqqNAS3lloE73BlQ)|[@richvdh](https://github.com/richvdh)||
|[1067](https://github.com/matrix-org/matrix-spec-proposals/issues/1067)|Spec @mentions|2017-11-14|2020-04-21|[1](https://docs.google.com/document/d/1oRhw3DJhbVAKkHAEgyt6ccV82wtXR_11qY7JqMcesUU/edit)|[@lukebarnard1](https://github.com/lukebarnard1)||
|[1033](https://github.com/matrix-org/matrix-spec-proposals/issues/1033)|Doc @room notifications|2017-10-23|2020-04-21|[1](https://docs.google.com/document/d/1qRdlg94cr9YXxPCwhW4HgI2oDrqQOUKX5HptZFBGf6o/edit#)|[@dbkr](https://github.com/dbkr)||
|[953](https://github.com/matrix-org/matrix-spec-proposals/issues/953)|Add /user_directory/search API|2017-07-18|2020-04-21|[1](https://docs.google.com/document/d/1Xc9lAM-FiIC66Z5pnaI4D5zqAqcFcZ5uHr3bYT-DWVk/edit)|[@erikjohnston](https://github.com/erikjohnston)||
|[910](https://github.com/matrix-org/matrix-spec-proposals/issues/910)|Add new Read Marker API to docs|2017-05-08|2020-04-21|[1](https://docs.google.com/document/d/1UWqdS-e1sdwkLDUY0wA4gZyIkRp-ekjsLZ8k6g_Zvso/edit)|[@lukebarnard1](https://github.com/lukebarnard1)||
|[855](https://github.com/matrix-org/matrix-spec-proposals/issues/855)|spec m.login.msisdn UI auth type|2017-03-24|2020-04-21|[1](https://docs.google.com/document/d/1B7q_3ruJzeQTg-uJHe1UScxbVLzgm451c25OjpYcojI/edit#)|[@dbkr](https://github.com/dbkr)||
|[829](https://github.com/matrix-org/matrix-spec-proposals/issues/829)|Need to spec msisdn login API|2017-03-08|2020-04-21|[1](https://docs.google.com/document/d/1-6ZSSW5YvCGhVFDyD2QExAUAdpCWjccvJT5xiyTTG2Y/edit#heading=h.79ot48krpkq7)|[@dbkr](https://github.com/dbkr)||
|[739](https://github.com/matrix-org/matrix-spec-proposals/issues/739)|Reporting inappropriate content in Matrix|2016-11-21|2020-04-21|[1](https://docs.google.com/document/d/15cUuF0VyBMtNIcyFqXvEmXsMURLgXzMOIW33qHoi89A/edit)|[@ara4n](https://github.com/ara4n)||
|[688](https://github.com/matrix-org/matrix-spec-proposals/issues/688)|Room Summaries (was: Calculate room names server-side)|2016-07-12|2022-03-01|[1](https://docs.google.com/document/d/11i14UI1cUz-OJ0knD5BFu7fmT6Fo327zvMYqfSAR7xs/edit#)|[@ara4n](https://github.com/ara4n)||
|[433](https://github.com/matrix-org/matrix-spec-proposals/issues/433)|Support for discovering API endpoints via .well-known URIs (SPEC-121)|2015-03-08|2022-03-01|[1](https://docs.google.com/document/d/1OdEj06qA7diURofyonIMgTR3fB_pWf12Txye41qd-U4/edit), [2](https://docs.google.com/document/d/1vF-uWlUYmf1Xo161m871H1upJbwiIPeikWGWzaE_lrU/edit#)|[@maxidor](https://github.com/maxidor), [@thers](https://github.com/thers)|[@uhoreg](https://github.com/uhoreg)|

### Postponed[](https://spec.matrix.org/proposals/#postponed)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[2190](https://github.com/matrix-org/matrix-spec-proposals/pull/2190)|MSC2190: Allow appservice bots to use /sync|2019-07-24|2022-11-03||[@tulir](https://github.com/tulir)||

### Abandoned[](https://spec.matrix.org/proposals/#abandoned)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[3978](https://github.com/matrix-org/matrix-spec-proposals/pull/3978)|MSC3978: Deprecate room tagging|2023-03-15|2023-03-16||[@uhoreg](https://github.com/uhoreg)||
|[3969](https://github.com/matrix-org/matrix-spec-proposals/pull/3969)|MSC3969: Size limits|2023-02-18|2023-09-30||[@progval](https://github.com/progval)||
|[3968](https://github.com/matrix-org/matrix-spec-proposals/pull/3968)|MSC3968: Poorer features|2023-02-18|2024-02-23||[@progval](https://github.com/progval)||
|[3852](https://github.com/matrix-org/matrix-spec-proposals/pull/3852)|MSC3852: Expose user agent information on `Device`|2022-07-22|2023-01-11||[@kerryarchibald](https://github.com/kerryarchibald)||
|[3790](https://github.com/matrix-org/matrix-spec-proposals/pull/3790)|MSC3790: Register Clients|2022-05-09|2022-05-11||[@xarvic](https://github.com/xarvic)||
|[3772](https://github.com/matrix-org/matrix-spec-proposals/pull/3772)|MSC3772: Push rule for mutually related events|2022-04-15|2022-10-07||[@clokep](https://github.com/clokep)||
|[3746](https://github.com/matrix-org/matrix-spec-proposals/pull/3746)|MSC3746: Render image data in reactions|2022-02-25|2023-07-06||[@AndrewRyanChama](https://github.com/AndrewRyanChama)||
|[3523](https://github.com/matrix-org/matrix-spec-proposals/pull/3523)|MSC3523: Timeboxed/ranged relations endpoint|2021-11-23|2023-06-01||[@turt2live](https://github.com/turt2live)||
|[3302](https://github.com/matrix-org/matrix-spec-proposals/pull/3302)|MSC3302: Stories via To-Device-Messaging|2021-07-31|2021-11-22||[@krille-chan](https://github.com/krille-chan)||
|[3265](https://github.com/matrix-org/matrix-spec-proposals/pull/3265)|MSC3265: Login and SSSS with a Single Password|2021-07-02|2022-12-12||[@cvwright](https://github.com/cvwright)||
|[3137](https://github.com/matrix-org/matrix-spec-proposals/pull/3137)|MSC3137: Define space room type, subset of MSC1772|2021-04-20|2021-04-29||[@t3chguy](https://github.com/t3chguy)||
|[3125](https://github.com/matrix-org/matrix-spec-proposals/pull/3125)|MSC3125: Limits API — Part 5: per-Instance limits|2021-04-15|2021-08-27||[@erkinalp](https://github.com/erkinalp)||
|[3074](https://github.com/matrix-org/matrix-spec-proposals/pull/3074)|MSC3074: Proposal for URIs conforming to RFC 3986 syntax.|2021-03-25|2021-03-29||[@alexmingoia](https://github.com/alexmingoia)||
|[3073](https://github.com/matrix-org/matrix-spec-proposals/pull/3073)|MSC3073: Role based access control|2021-03-25|2023-09-19||[@erkinalp](https://github.com/erkinalp)||
|[3068](https://github.com/matrix-org/matrix-spec-proposals/pull/3068)|[WIP] MSC3068: Compliance tiers|2021-03-18|2021-03-23||[@erkinalp](https://github.com/erkinalp)||
|[3053](https://github.com/matrix-org/matrix-spec-proposals/pull/3053)|MSC3053: Limits API — Part 2: per-Room limits|2021-03-07|2021-08-27||[@erkinalp](https://github.com/erkinalp)||
|[3007](https://github.com/matrix-org/matrix-spec-proposals/pull/3007)|MSC3007: Forced insertion and room blocking by self-banning|2021-02-14|2021-02-20||[@erkinalp](https://github.com/erkinalp)||
|[3006](https://github.com/matrix-org/matrix-spec-proposals/pull/3006)|MSC3006: Bot Interactions|2021-02-13|2023-10-27||[@MTRNord](https://github.com/MTRNord)||
|[3005](https://github.com/matrix-org/matrix-spec-proposals/pull/3005)|MSC3005: Streaming Federation Events|2021-02-10|2022-10-21||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[2912](https://github.com/matrix-org/matrix-spec-proposals/pull/2912)|MSC2912: Setting cross-signing keys during registration|2020-12-15|2020-12-17||[@uhoreg](https://github.com/uhoreg)||
|[2880](https://github.com/matrix-org/matrix-spec-proposals/pull/2880)|MSC2880: extend event and state API to allow sending to all rooms|2020-11-27|2021-08-27||[@ghost](https://github.com/ghost)||
|[2875](https://github.com/matrix-org/matrix-spec-proposals/pull/2875)|MSC2875: room descriptions|2020-11-23|2021-08-27||[@ghost](https://github.com/ghost)||
|[2834](https://github.com/matrix-org/matrix-spec-proposals/pull/2834)|MSC2834: Media IDs as hashes|2020-10-26|2021-08-27||[@ghost](https://github.com/ghost)||
|[2810](https://github.com/matrix-org/matrix-spec-proposals/pull/2810)|MSC2810: Consistent globs specification|2020-10-08|2022-03-01||[@turt2live](https://github.com/turt2live)||
|[2773](https://github.com/matrix-org/matrix-spec-proposals/pull/2773)|MSC2773: Room kinds|2020-09-11|2021-02-27||[@turt2live](https://github.com/turt2live)||
|[2771](https://github.com/matrix-org/matrix-spec-proposals/pull/2771)|MSC2771: Bookmarks|2020-09-10|2022-05-23||[@MTRNord](https://github.com/MTRNord)||
|[2704](https://github.com/matrix-org/matrix-spec-proposals/pull/2704)|MSC2704: Explicitly allow alternative origins in MXC URIs and specify deduplication requirements on uploads|2020-07-28|2023-12-16||[@turt2live](https://github.com/turt2live)||
|[2638](https://github.com/matrix-org/matrix-spec-proposals/pull/2638)|MSC2638: Ability for clients to request homeservers to resync device lists|2020-06-15|2024-03-02||[@babolivier](https://github.com/babolivier)||
|[2631](https://github.com/matrix-org/matrix-spec-proposals/pull/2631)|MSC2631: Add `default_payload` to PusherData|2020-06-12|2020-07-07||[@ismailgulek](https://github.com/ismailgulek)||
|[2625](https://github.com/matrix-org/matrix-spec-proposals/pull/2625)|MSC2625: Add `mark_unread` push rule action|2020-06-10|2020-07-01||[@richvdh](https://github.com/richvdh)||
|[2579](https://github.com/matrix-org/matrix-spec-proposals/pull/2579)|MSC2579: Improved tagging support|2020-05-25|2020-07-24||[@turt2live](https://github.com/turt2live)||
|[2376](https://github.com/matrix-org/matrix-spec-proposals/pull/2376)|MSC2376: Disable URL Previews|2019-12-03|2021-01-14||[@Sorunome](https://github.com/Sorunome)||
|[2314](https://github.com/matrix-org/matrix-spec-proposals/pull/2314)|MSC2314: A Method to Backfill Room State|2019-10-07|2022-04-14||[@hawkowl](https://github.com/hawkowl)||
|[2233](https://github.com/matrix-org/matrix-spec-proposals/pull/2233)|MSC2233: Unauthenticated Capabilities API|2019-08-15|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2000](https://github.com/matrix-org/matrix-spec-proposals/pull/2000)|MSC2000: Server-side password policies|2019-05-15|2023-02-27||[@babolivier](https://github.com/babolivier)||
|[1935](https://github.com/matrix-org/matrix-spec-proposals/pull/1935)|MSC1935: Key validity enforcement|2019-03-20|2020-04-20||[@turt2live](https://github.com/turt2live)||
|[1920](https://github.com/matrix-org/matrix-spec-proposals/pull/1920)|MSC1920: Alternative texts for stickers|2019-03-08|2024-03-02||[@babolivier](https://github.com/babolivier)||
|[1722](https://github.com/matrix-org/matrix-spec-proposals/pull/1722)|MSC1722: Support for displaying math(s) in messages|2018-11-15|2020-10-19||[@uhoreg](https://github.com/uhoreg)||
|[1714](https://github.com/matrix-org/matrix-spec-proposals/pull/1714)|MSC1714: using the TLS private key to sign federation-signing keys|2018-11-09|2022-03-01||[@richvdh](https://github.com/richvdh)||
|[1700](https://github.com/matrix-org/matrix-spec-proposals/pull/1700)|MSC1700: Improving .well-known discovery|2018-10-19|2022-04-14||[@turt2live](https://github.com/turt2live)||
|[1680](https://github.com/matrix-org/matrix-spec-proposals/issues/1680)|MSC1680: cross-signing of devices to simplify key verification|2018-09-20|2020-04-20|[1](https://github.com/matrix-org/matrix-doc/pull/1681)|[@uhoreg](https://github.com/uhoreg)||
|[1489](https://github.com/matrix-org/matrix-spec-proposals/pull/1489)|Suggested Mentions Proposal|2018-08-06|2020-04-21||[@Half-Shot](https://github.com/Half-Shot)||
|[1217](https://github.com/matrix-org/matrix-spec-proposals/issues/1217)|Visibility of groups to group members and their non-members|2018-05-10|2020-08-21|[1](https://docs.google.com/document/d/13OQ0gtdLsha4RKttxVZlGTKEncvjOToa2duv8bOdyvs/edit#heading=h.xsf65cn5ty5q)|[@lampholder](https://github.com/lampholder)||
|[1213](https://github.com/matrix-org/matrix-spec-proposals/issues/1213)|Set defaults for m.federate|2018-05-10|2020-08-26|[1](https://docs.google.com/document/d/14zqsbwl5KKil-bB8w2HMhidBVmFkP9Q7EQKFwKIIfZc/edit#heading=h.eipip5qhqo0d)|[@psaavedra](https://github.com/psaavedra)||
|[1209](https://github.com/matrix-org/matrix-spec-proposals/issues/1209)|Server Side Profile API|2018-05-10|2020-08-21|[1](https://docs.google.com/document/d/18r84a3IgsItUu1k326XZCGHbVy0S-YLqrfvItGaEo_4/edit#heading=h.oxxmp054yga2)|[@erikjohnston](https://github.com/erikjohnston)||
|[531](https://github.com/matrix-org/matrix-spec-proposals/issues/531)|Homeservers as OAuth authorization endpoints (resource owners) (SPEC-206)|2015-07-25|2020-08-21|[1](https://docs.google.com/document/d/1vEPFlX79oa1foBmar6i8nvw-hB4SXfVqg6o6Wsdl1kQ/edit)|[@Kegsay](https://github.com/Kegsay)||

### Obsolete[](https://spec.matrix.org/proposals/#obsolete)

|MSC|Title|Created at|Updated at|Docs|Author|Shepherd|
|---|---|---|---|---|---|---|
|[4123](https://github.com/matrix-org/matrix-spec-proposals/pull/4123)|MSC4123: Allow `knock` -> `join` transition|2024-04-06|2024-05-18||[@Kladki](https://github.com/Kladki)||
|[4052](https://github.com/matrix-org/matrix-spec-proposals/pull/4052)|MSC4052: Hiding read receipts UI in certain rooms|2023-09-06|2023-10-06||[@cloudrac3r](https://github.com/cloudrac3r)||
|[4037](https://github.com/matrix-org/matrix-spec-proposals/pull/4037)|MSC4037: Thread root is not in the thread|2023-07-13|2023-11-23||[@andybalaam](https://github.com/andybalaam)||
|[4018](https://github.com/matrix-org/matrix-spec-proposals/pull/4018)|MSC4018: Reliable call membership|2023-05-19|2024-09-13||[@robintown](https://github.com/robintown)||
|[3910](https://github.com/matrix-org/matrix-spec-proposals/pull/3910)|MSC3910: Content tokens for media|2022-10-14|2022-11-04||[@richvdh](https://github.com/richvdh)||
|[3906](https://github.com/matrix-org/matrix-spec-proposals/pull/3906)|MSC3906: Mechanism to allow sign in and E2EE set up via QR code|2022-10-13|2024-04-30||[@hughns](https://github.com/hughns)||
|[3903](https://github.com/matrix-org/matrix-spec-proposals/pull/3903)|MSC3903: X25519 Elliptic-curve Diffie-Hellman ephemeral for establishing secure channel between two Matrix clients|2022-10-04|2024-04-30||[@hughns](https://github.com/hughns)||
|[3886](https://github.com/matrix-org/matrix-spec-proposals/pull/3886)|MSC3886: Simple client rendezvous capability|2022-09-07|2024-04-30||[@hughns](https://github.com/hughns)||
|[3862](https://github.com/matrix-org/matrix-spec-proposals/pull/3862)|MSC3862: event_match (almost) anything|2022-08-05|2023-02-06||[@Johennes](https://github.com/Johennes)||
|[3859](https://github.com/matrix-org/matrix-spec-proposals/pull/3859)|MSC3859: Add well known media domain proposal|2022-08-04|2022-08-17||[@Fizzadar](https://github.com/Fizzadar)||
|[3760](https://github.com/matrix-org/matrix-spec-proposals/pull/3760)|MSC3760: State sub-keys|2022-03-31|2022-04-13||[@andybalaam](https://github.com/andybalaam)||
|[3613](https://github.com/matrix-org/matrix-spec-proposals/pull/3613)|MSC3613: Combinatorial join rules|2021-12-31|2022-05-04||[@turt2live](https://github.com/turt2live)||
|[3585](https://github.com/matrix-org/matrix-spec-proposals/pull/3585)|MSC3585: Allow the base event to be omitted from `/federation/v1/event_auth` response|2021-12-22|2022-01-06||[@richvdh](https://github.com/richvdh)||
|[3517](https://github.com/matrix-org/matrix-spec-proposals/pull/3517)|MSC3517: "Mention" Pushrule|2021-11-21|2023-01-08||[@ShadowJonathan](https://github.com/ShadowJonathan)||
|[3510](https://github.com/matrix-org/matrix-spec-proposals/pull/3510)|MSC3510: Let users kick/ban/demote each other|2021-11-20|2022-10-24||[@ara4n](https://github.com/ara4n)||
|[3464](https://github.com/matrix-org/matrix-spec-proposals/pull/3464)|MSC3464: Allow Users to Post on Behalf of Other Users|2021-10-29|2024-07-29||[@sumnerevans](https://github.com/sumnerevans)||
|[3429](https://github.com/matrix-org/matrix-spec-proposals/pull/3429)|MSC3429: Individual room preview API|2021-10-05|2022-07-19||[@turt2live](https://github.com/turt2live)||
|[3306](https://github.com/matrix-org/matrix-spec-proposals/pull/3306)|MSC3306: Counting unread messages|2021-08-02|2021-08-03||[@KitsuneRal](https://github.com/KitsuneRal)||
|[3286](https://github.com/matrix-org/matrix-spec-proposals/pull/3286)|MSC3286: Media spoilers|2021-07-18|2022-02-14||[@robintown](https://github.com/robintown)||
|[3282](https://github.com/matrix-org/matrix-spec-proposals/pull/3282)|MSC3282: Expose enable_set_displayname in capabilities response|2021-07-13|2021-07-19||[@JonasKress](https://github.com/JonasKress)||
|[3279](https://github.com/matrix-org/matrix-spec-proposals/pull/3279)|MSC3279: Expose enable_set_displayname in capabilities response|2021-07-13|2021-07-13||[@JonasKress](https://github.com/JonasKress)||
|[3270](https://github.com/matrix-org/matrix-spec-proposals/pull/3270)|MSC3270: Symmetric megolm backup|2021-07-07|2024-03-08||[@uhoreg](https://github.com/uhoreg)||
|[3226](https://github.com/matrix-org/matrix-spec-proposals/pull/3226)|MSC3226: Per-room spell check|2021-05-31|2021-08-30||[@afranke](https://github.com/afranke)||
|[3067](https://github.com/matrix-org/matrix-spec-proposals/pull/3067)|MSC3067: Prevent/remove legacy groups from being in the spec|2021-03-18|2021-05-01||[@turt2live](https://github.com/turt2live)||
|[2876](https://github.com/matrix-org/matrix-spec-proposals/pull/2876)|MSC2876: Allowing widgets to read events in a room|2020-11-24|2021-05-14||[@turt2live](https://github.com/turt2live)||
|[2775](https://github.com/matrix-org/matrix-spec-proposals/pull/2775)|MSC2775: Lazy loading over federation|2020-09-14|2022-10-18||[@ara4n](https://github.com/ara4n)||
|[2703](https://github.com/matrix-org/matrix-spec-proposals/pull/2703)|MSC2703: Media ID grammar|2020-07-28|2022-10-12||[@turt2live](https://github.com/turt2live)||
|[2700](https://github.com/matrix-org/matrix-spec-proposals/pull/2700)|MSC2700: Thumbnail requirements for the media repo|2020-07-28|2023-12-16||[@turt2live](https://github.com/turt2live)||
|[2589](https://github.com/matrix-org/matrix-spec-proposals/pull/2589)|MSC2589: Improve replies|2020-05-29|2022-03-01||[@turt2live](https://github.com/turt2live)||
|[2516](https://github.com/matrix-org/matrix-spec-proposals/pull/2516)|MSC2516: Add a new message type for voice messages|2020-04-27|2021-12-27||[@ludwigbald](https://github.com/ludwigbald)||
|[2475](https://github.com/matrix-org/matrix-spec-proposals/pull/2475)|MSC2475: API versioning|2020-03-27|2021-09-20||[@turt2live](https://github.com/turt2live)||
|[2461](https://github.com/matrix-org/matrix-spec-proposals/pull/2461)|MSC2461: Proposal for Authenticated Content Repository API|2020-03-15|2024-05-15||[@SyrupThinker](https://github.com/SyrupThinker)||
|[2390](https://github.com/matrix-org/matrix-spec-proposals/pull/2390)|MSC2390: On the EDU-to-PDU transition.|2019-12-18|2022-04-26||[@jevolk](https://github.com/jevolk)||
|[2389](https://github.com/matrix-org/matrix-spec-proposals/pull/2389)|MSC2389: Toward the EDU-to-PDU transition: Typing.|2019-12-18|2022-04-26||[@jevolk](https://github.com/jevolk)||
|[2261](https://github.com/matrix-org/matrix-spec-proposals/pull/2261)|MSC2261: Allow `m.room.aliases` events to be redacted by room admins|2019-08-29|2020-04-20||[@richvdh](https://github.com/richvdh)||
|[2260](https://github.com/matrix-org/matrix-spec-proposals/pull/2260)|MSC2260: Update the auth rules for `m.room.aliases` events|2019-08-29|2022-03-01||[@richvdh](https://github.com/richvdh)||
|[2229](https://github.com/matrix-org/matrix-spec-proposals/pull/2229)|MSC2229: Allowing 3PID Owners to Rebind|2019-08-12|2020-04-20||[@anoadragon453](https://github.com/anoadragon453)||
|[2153](https://github.com/matrix-org/matrix-spec-proposals/pull/2153)|MSC2153: Add a default push rule to ignore m.reaction events|2019-07-04|2022-07-31||[@jryans](https://github.com/jryans)||
|[1958](https://github.com/matrix-org/matrix-spec-proposals/pull/1958)|MSC1958: Widget architecture changes|2019-04-09|2024-05-14||[@turt2live](https://github.com/turt2live)||
|[1951](https://github.com/matrix-org/matrix-spec-proposals/pull/1951)|MSC1951: Custom sticker packs and emoji (mk II)|2019-04-04|2024-05-13||[@turt2live](https://github.com/turt2live)||
|[1943](https://github.com/matrix-org/matrix-spec-proposals/pull/1943)|MSC1943: Set default room as v3|2019-03-27|2020-04-20||[@neilisfragile](https://github.com/neilisfragile)||
|[1902](https://github.com/matrix-org/matrix-spec-proposals/pull/1902)|MSC1902: Split the media repo into s2s and c2s parts|2019-02-24|2023-11-25||[@turt2live](https://github.com/turt2live)||
|[1888](https://github.com/matrix-org/matrix-spec-proposals/pull/1888)|[WIP] MSC1888: Proposal to send EDUs to appservices|2019-02-15|2022-03-01||[@Half-Shot](https://github.com/Half-Shot)||
|[1849](https://github.com/matrix-org/matrix-spec-proposals/pull/1849)|MSC1849: Proposal for m.relates_to aggregations|2019-02-04|2022-11-08||[@ara4n](https://github.com/ara4n)||
|[1777](https://github.com/matrix-org/matrix-spec-proposals/pull/1777)|[WIPish] MSC1777: peeking over federation (via server pseudousers)|2019-01-06|2020-10-28||[@ara4n](https://github.com/ara4n)||
|[1776](https://github.com/matrix-org/matrix-spec-proposals/pull/1776)|[WIPish] MSC1776: Implementing peeking via /sync|2019-01-06|2020-09-05||[@ara4n](https://github.com/ara4n)||
|[1731](https://github.com/matrix-org/matrix-spec-proposals/pull/1731)|MSC1731: Mechanism for redirecting to an alternative server during SSO login|2018-11-24|2020-04-20||[@richvdh](https://github.com/richvdh)||
|[1703](https://github.com/matrix-org/matrix-spec-proposals/pull/1703)|MSC1703: encrypting recovery keys for online megolm backups|2018-10-23|2021-06-08||[@dbkr](https://github.com/dbkr)||
|[3793](https://github.com/matrix-org/matrix-spec-proposals/issues/3793)|Future Proposal for Cryptographic Challenge Login Flow and E2E Key Backup Recovery|2018-10-18|2022-07-19|[1](https://docs.google.com/document/d/1QXMJmLiRWQxZOlynVVk99KhumlM7-7bTrGatBG0AQfU/edit#heading=h.apnhwogtw3jb)|[@ara4n](https://github.com/ara4n)||
|[1695](https://github.com/matrix-org/matrix-spec-proposals/pull/1695)|MSC1695 Message Edits|2018-10-12|2022-03-01||[@Half-Shot](https://github.com/Half-Shot)||
|[1687](https://github.com/matrix-org/matrix-spec-proposals/issues/1687)|MSC1687: Encrypting recovery keys for online megolm backups|2018-09-26|2022-11-08|[1](https://github.com/matrix-org/matrix-spec-proposals/pull/1686)|[@ara4n](https://github.com/ara4n)||
|[1421](https://github.com/matrix-org/matrix-spec-proposals/issues/1421)|Make a Discourse forum the canonical location for spec discussions|2018-07-16|2020-04-21|[1](https://docs.google.com/document/d/1EKe135LPtpLcdAoEjwIDMKoIBdq1Ev2I6hDbkTplbh0/edit?usp=sharing)|[@non-Jedi](https://github.com/non-Jedi)||
|[1410](https://github.com/matrix-org/matrix-spec-proposals/issues/1410)|Proposal for Rich Bridging|2018-07-12|2020-04-21|[1](https://docs.google.com/document/d/1uBxekDnl9WBPWA989W1_QeOWA560TZI27g-Xju9Uk3w/edit#)|[@Half-Shot](https://github.com/Half-Shot)||
|[1324](https://github.com/matrix-org/matrix-spec-proposals/issues/1324)|Custom protocol definitions for application services|2018-06-20|2018-08-30|[1](https://docs.google.com/document/d/1nCAGu9ZDqum39yj1-pun-sldCCHgeDA0tPqC94R4Qcg/edit?usp=sharing)|[@anoadragon453](https://github.com/anoadragon453)||
|[3808](https://github.com/matrix-org/matrix-spec-proposals/issues/3808)|Mechanism to communicate 3PID binding updates or deletions to homeservers|2018-06-20|2023-05-13|[1](https://docs.google.com/document/d/1LPHBfJQ6x5iOkm2_ZYPNHNjyAzJ4hWNdOGcPGpnEb14)|[@babolivier](https://github.com/babolivier)||
|[1318](https://github.com/matrix-org/matrix-spec-proposals/issues/1318)|Proposal for Open Governance of Matrix.org|2018-06-20|2020-04-21|[1](https://docs.google.com/document/d/1vASIt2v0UwKnUKHNEjA_P3SFCM0kcIUl1LQyM--ax04/edit#)|[@ara4n](https://github.com/ara4n)||
|[1310](https://github.com/matrix-org/matrix-spec-proposals/issues/1310)|Proposal for a media information API|2018-06-17|2022-03-01|[1](https://docs.google.com/document/d/1yNnOIFuYYu33YVVhOMPZoNQ8YBGcSfSiO1bTe1NwPHI/edit?usp=sharing)|[@turt2live](https://github.com/turt2live)||
|[1308](https://github.com/matrix-org/matrix-spec-proposals/issues/1308)|Proposal for speeding up review of simple spec changes|2018-06-14|2022-05-09||[@ara4n](https://github.com/ara4n)||
|[3805](https://github.com/matrix-org/matrix-spec-proposals/issues/3805)|Proposal for improving authorization for the matrix profile API|2018-06-13|2024-08-14|[1](https://docs.google.com/document/d/1G7JjyTuJlZHieuAflGFWmdKyNViGGLRTWON7AMl0wrM/edit#)|[@turt2live](https://github.com/turt2live)||
|[3804](https://github.com/matrix-org/matrix-spec-proposals/issues/3804)|Mechanisms for communicating erasure requests to bots and federated homeservers|2018-06-05|2022-07-19|[1](https://docs.google.com/document/d/17ssplT4pX80ebmyaFIYcXtINV88lBT83ddW9ZhjsDnI)|[@richvdh](https://github.com/richvdh)||
|[1270](https://github.com/matrix-org/matrix-spec-proposals/issues/1270)|Proposal Add /_matrix/media/v1/resolve_url to Client-Server API: download and preview urls in the clients despite CORS|2018-05-31|2022-03-01|[1](https://docs.google.com/document/d/1bbX1yxNETmMa-AxBGjIpb4lNoTuc3vjGXmbZWrNBlzM/edit?usp=sharing)|[@oivoodoo](https://github.com/oivoodoo)||
|[1256](https://github.com/matrix-org/matrix-spec-proposals/issues/1256)|Custom emoji and sticker packs in matrix|2018-05-23|2021-07-28|[1](https://docs.google.com/document/d/1zHS14unA2Wb3DgTL5fiymlWKZo4WMJpmmJOgY_2g6pg/edit?usp=sharing)|[@turt2live](https://github.com/turt2live)||
|[1231](https://github.com/matrix-org/matrix-spec-proposals/issues/1231)|Handling Consent for Privacy Policy|2018-05-10|2022-03-01|[1](https://docs.google.com/document/d/1-Q_Z9dD3VTfsNYtK_WTzyTQR4HQWsntt-_DwgoW02ZU/edit#heading=h.cvd8uae3gmto)|[@neilisfragile](https://github.com/neilisfragile)||
|[1226](https://github.com/matrix-org/matrix-spec-proposals/issues/1226)|State Reset mitigation proposal|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1L2cr8djdpOFXJgqGTf3gUrk-YGBYf--rP8Nw6mYYAu8/edit#heading=h.vazyvubo3b4z)|[@richvdh](https://github.com/richvdh)||
|[1225](https://github.com/matrix-org/matrix-spec-proposals/issues/1225)|Extensible event types & fallback in Matrix|2018-05-10|2019-01-03|[1](https://docs.google.com/document/d/1FUVFzTOF4a6XBKVTk55bVRIk4N9u5uZkHS4Rjr_SGxo/edit#heading=h.m568t57r6od9)|[@ara4n](https://github.com/ara4n)||
|[1224](https://github.com/matrix-org/matrix-spec-proposals/issues/1224)|Replies - next steps|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1FZsvodn2C0iKJDtn-8y8IPwOa96ixoJejK3gMLVOXHM/edit)|[@t3chguy](https://github.com/t3chguy)||
|[1223](https://github.com/matrix-org/matrix-spec-proposals/issues/1223)|Replies event format|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1KLdKtuZBbFoWDSfN4KM3p7LnIvFBQfSORICBo8zRHaE/edit)|[@t3chguy](https://github.com/t3chguy)||
|[1222](https://github.com/matrix-org/matrix-spec-proposals/issues/1222)|Pushing updates about Groups (Communities) to clients|2018-05-10|2021-04-11|[1](https://drive.google.com/open?id=1GzwhGdnWWMENYOaXMFP8CD-M9ny1vznxHnNqT3I3NZI)|[@ara4n](https://github.com/ara4n)||
|[1220](https://github.com/matrix-org/matrix-spec-proposals/issues/1220)|Rich quoting proposal|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/146zJr4h6odczMeGUH99dxDZk0i_iVtDiVMy510G25jI/edit)|[@t3chguy](https://github.com/t3chguy)||
|[1215](https://github.com/matrix-org/matrix-spec-proposals/issues/1215)|Groups as Rooms|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1ZnAuA_zti-K2-RnheXII1F1-oyVziT4tJffdw1-SHrE/edit#)|[@ara4n](https://github.com/ara4n)||
|[1214](https://github.com/matrix-org/matrix-spec-proposals/issues/1214)|Related Groups (i.e. flair)|2018-05-10|2021-05-01|[1](https://docs.google.com/document/d/1wCLXwUT3r4gVFuQpwWMHxl-nEf_Kx2pv34vZQQVb_Bc/edit#heading=h.82i09uxamcfq)|[@lukebarnard1](https://github.com/lukebarnard1)||
|[1202](https://github.com/matrix-org/matrix-spec-proposals/issues/1202)|Profile Personae|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1_15r2b43506FhgEKjLZC32BxRy6JAlB8ayCazMR0_S0/edit)|[@erikjohnston](https://github.com/erikjohnston)||
|[1196](https://github.com/matrix-org/matrix-spec-proposals/issues/1196)|Matrix Escape Hatch (Versioned Rooms)|2018-05-10|2020-04-21|[1](https://docs.google.com/document/d/1_N9HhXEqO9yX1c4TSlVAAvTaiyzDXTuVmGW-3hJe840/edit#heading=h.83j3cb3h3i4c)|[@leonerd](https://github.com/leonerd)||
|[1194](https://github.com/matrix-org/matrix-spec-proposals/issues/1194)|A way for HSes to remove bindings from ISes (aka unbind)|2018-05-09|2020-04-21|[1](https://docs.google.com/document/d/1uDY__XA87VZDnJAzewfxGEVKxLW0KgDWg2oPqM4Ej80/edit#)|[@dbkr](https://github.com/dbkr)||
|[1116](https://github.com/matrix-org/matrix-spec-proposals/pull/1116)|Spec a format for calendar events in rooms.|2018-02-06|2022-03-02||[@Half-Shot](https://github.com/Half-Shot)||
|[971](https://github.com/matrix-org/matrix-spec-proposals/issues/971)|Add groups stuff to spec|2017-08-10|2022-11-08||[@erikjohnston](https://github.com/erikjohnston)||
|[441](https://github.com/matrix-org/matrix-spec-proposals/issues/441)|Support for Reactions / Aggregations|2015-03-29|2020-04-21|[1](https://docs.google.com/document/d/1CnNbYSSea0KcyhEI6-rB8R8u6DCZyZv-Pv4hhoXJHSE/edit)|[@pik](https://github.com/pik)|[@ara4n](https://github.com/ara4n)|
|[455](https://github.com/matrix-org/matrix-spec-proposals/issues/455)|Do we want to specify a matrix:// URI scheme for rooms? (SPEC-5)|2014-09-15|2020-04-21|[1](https://docs.google.com/document/d/18A3ZRgGR-GLlPXF_VIHxywWiX1vpMvNfAU6JCnNMVuQ/edit)|[@KitsuneRal](https://github.com/KitsuneRal)||