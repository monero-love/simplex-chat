# Keyoxide address verifier interoperability work

This private development import supports a standalone verifier in the companion [Keyoxide work branch](https://github.com/monero-love/keyoxide-web/tree/work/simplex-address-verifier/experiments/simplex-verifier). The verifier implementation and tests live there to avoid maintaining two copies. This branch does not change the SimpleX engine or deploy a bot.

The initial verifier was tested against SimpleX CLI v7.1.4.0 and source commit `cb4b50db169a2638c78e0b41730d5759c559b473`. The source upstream is <https://github.com/simplex-chat/simplex-chat>. The original stable branch remains unchanged.

## Tested contract

1. A valid OpenPGP UID self-certification must claim the exact full reusable contact address. Neither an unsigned address nor a display name is proof.
2. `APIConnectPlan` must report a fresh contact address. Invitations, group links, business connections and existing-contact shortcuts are rejected. The returned full link must match the signed claim exactly.
3. `APIConnect` must return `SentInvitation`. Its pending connection ID is matched to the `ContactConnected` event's active connection ID and local user ID.
4. A versioned challenge binds the exact address, primary fingerprint, verifier identity, random nonce and expiry. It is sent to that direct contact using `APISendMessages`.
5. The operator reviews and signs the challenge locally with GnuPG. Only a detached signature received in the same direct user/contact/connection is considered. Valid signing subkeys are supported.
6. The nonce is consumed once, including failed response attempts. Expiry, missing response or failed transport never produce a checked result.

The standalone automated suite now passes 35 tests, with mocked chat transport and signed-claim revalidation of the browser request envelope. The Keyoxide suite passes 45 tests, plus Chromium desktop and mobile tests. A separate live smoke test created two disposable local profiles, connected only those profiles through their address, exchanged a challenge and a GnuPG signature, verified it and rejected replay. Its keys and chat databases were removed. No transcripts or live contact addresses are committed.

## Next integration boundary

Keep any future API compatibility changes small and covered by the verifier's adapter tests. No SimpleX core modification is currently needed. Address rotation requires a new connection check, because an existing chat can outlive a deleted address. Private addresses require explicit opt-in and must not be published by default.

Success means that this verifier reached a contact through the claimed address and received a fresh signature from the claimed key. It is not independently verifiable evidence of chat delivery. Any future public badge must be a separate verifier attestation labeled “SimpleX address checked by monero.love on [date]”. No badge or public endpoint exists in this pass.

For local execution and manual signing instructions, see the companion README. Its live-test script is explicit and opt-in. Do not run it on page load or against other users.

## Implemented Keyoxide handoff

The companion development branch now displays SimpleX claims as **Not checked**, suppresses generic web-provider verification in the browser and API, and offers an explicit opt-in download of the exact address and signed public-key profile. The user invokes `node cli.js --request ...` locally. The CLI independently verifies the claim before opening its loopback WebSocket. Saving the request does not contact a SimpleX user or create a public badge.

No SimpleX core change is needed for this flow. See [build, preview and validation instructions](https://github.com/monero-love/keyoxide-web/tree/work/simplex-address-verifier/docs/simplex-verifier.md).

The companion release candidate pins patched OpenPGP and WebSocket dependencies, bounds transport input and queues, and uses private, durable one-use state with sensitive-record deletion. The two-disposable-profile live test was rerun on 2026-11-21. See its release audit for supported runtime and container checks. This branch still makes no SimpleX engine changes.
