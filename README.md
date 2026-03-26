# Ciphersky (A fork of Bluesky Social)

Welcome friends! This is the codebase for __Ciphersky__, a fork of the official Bluesky Social app that adds "private" posts via encryption.

## Disclaimer

I am not a cybersecurity or cryptography expert. This application does not promise to encrypt or decrypt data in any secure or invulnerable method. You should expect your "private" posts to be easily decrypted by others.

ATProto is entirely public by design; this app will use your ATProto repo to store some necessary public app data to make encryption and permission control work (your public key and your "allowed" list of users, respectively).

Do *not* use for any real secrets!

## The Goal

This project aims to let you to manage an internal "allowed" list of users who will have access to your "private" posts. Users of Ciphersky who are on your list will see the post content, while other users will not see the post at all. Users of the official Bluesky Social app or other third-party clients (even if they are on your "allowed" list) will see only the meaningless garbled encrypted message.

## The Plan

- [ ] __Encryption Setup__: When you login for the first time, the app generates an X25519 keypair. The private key is stored locally on your device in Android Keystore, and your public key is posted in a record on your ATProto repo (in a separate namespace from Bluesky data, don't worry). This is used by AES for as the key for encryption and decryption.
- [ ] __Permission Control__: Other users of Ciphersky can be added to your "allowed" user list. This will allow them to decrypt and read your "private" posts. Doing this will compute a Shared Key and post an HKDF-wrapped AES content key to your public "allowed" list key bundle.
- [ ] __Permission Discovery__: When the added user discovers they've been added to your list, they compute the same Shared Key and cache it locally for decryption.
  - *Implementation Note*: I'm debating active scanning of followed users vs lazy key discovery. Will likely start with the former and later implement the latter.
- [ ] __Permission Revocation__: Revoking a user from your "allowed" list will remove them from your key bundle, and the app on the other user's end will detect this removal and delete their Shared Key from cache. Since they technically could still have a copy of the key elsewhere, Encryption Setup is performed again and a new version of your key is posted *for future posts*. New encrypted posts will use this version of your key, and people on your allowed list will recompute their Shared Key.
  - *Security Note*: Be aware that your existing posts are encrypted with the previous version, and that revoked users may still have a copy of the Shared Key elsewhere! This means that __*revoking access from a user does not securely revoke their access to your existing posts,*__ only future posts.
    - I am exploring the possibility of editing ATProto records in order to re-encrypt existing posts for complete, secure revocation.
- [ ] __Post Composer Encryption Toggle__: The Post Composer will have an additional toggle labeled "Encrypt Post". Posting with this toggle on will take your original message and encrypt it using AES-256-GCM with your public key, then publicly post the encrypted text with a signifier Unicode character at the beginning to mark it as an encrypted Ciphersky message.
- [ ] __Reading Encrypted Posts__: When the Ciphersky client sees the Unicode signifier at the beginning of a post, it knows to use your Shared Key you have with the post's author to decrypt it and reveal the message.




<br>

---

# Bluesky Social's original README

Get the official Bluesky Social app:

- **Web: [bsky.app](https://bsky.app)**
- **iOS: [App Store](https://apps.apple.com/us/app/bluesky-social/id6444370199)**
- **Android: [Play Store](https://play.google.com/store/apps/details?id=xyz.blueskyweb.app)**

## Development Resources

This is a [React Native](https://reactnative.dev/) application, written in the TypeScript programming language. It builds on the `atproto` TypeScript packages (like [`@atproto/api`](https://www.npmjs.com/package/@atproto/api)), which are also open source, but in [a different git repository](https://github.com/bluesky-social/atproto).

There is a small amount of Go language source code (in `./bskyweb/`), for a web service that returns the React Native Web application.

The [Build Instructions](./docs/build.md) are a good place to get started with the app itself.

The Authenticated Transfer Protocol ("AT Protocol" or "atproto") is a decentralized social media protocol. You don't *need* to understand AT Protocol to work with this application, but it can help. Learn more at:

- [Overview and Guides](https://atproto.com/guides/overview)
- [GitHub Discussions](https://github.com/bluesky-social/atproto/discussions) 👈 Great place to ask questions
- [Protocol Specifications](https://atproto.com/specs/atp)
- [Blogpost on self-authenticating data structures](https://bsky.social/about/blog/3-6-2022-a-self-authenticating-social-protocol)

The Bluesky Social application encompasses a set of schemas and APIs built in the overall AT Protocol framework. The namespace for these "Lexicons" is `app.bsky.*`.

## Contributions

> [!NOTE]
> While we do accept contributions, we prioritize high quality issues and pull requests. Adhering to the below guidelines will ensure a more timely review.

**Rules:**

- We may not respond to your issue or PR.
- We may close an issue or PR without much feedback.
- We may lock discussions or contributions if our attention is getting DDOSed.
- We're not going to provide support for build issues.

**Guidelines:**

- Check for existing issues before filing a new one please.
- Open an issue and give some time for discussion before submitting a PR.
- Stay away from PRs like...
  - Changing "Post" to "Skeet."
  - Refactoring the codebase, e.g., to replace React Query with Redux Toolkit or something.
  - Adding entirely new features without prior discussion. 

Remember, we serve a wide community of users. Our day-to-day involves us constantly asking "which top priority is our top priority." If you submit well-written PRs that solve problems concisely, that's an awesome contribution. Otherwise, as much as we'd love to accept your ideas and contributions, we really don't have the bandwidth. That's what forking is for!

## Forking guidelines

You have our blessing 🪄✨ to fork this application! However, it's very important to be clear to users when you're giving them a fork.

Please be sure to:

- Change all branding in the repository and UI to clearly differentiate from Bluesky.
- Change any support links (feedback, email, terms of service, etc) to your own systems.
- Replace any analytics or error-collection systems with your own so we don't get super confused.

## Security disclosures

If you discover any security issues, please send an email to security@bsky.app. The email is automatically CC'd to the entire team and we'll respond promptly.

## Are you a developer interested in building on atproto?

Bluesky is an open social network built on the AT Protocol, a flexible technology that will never lock developers out of the ecosystems that they help build. With atproto, third-party integration can be as seamless as first-party through custom feeds, federated services, clients, and more.

## License (MIT)

See [./LICENSE](./LICENSE) for the full license.

Bluesky Social PBC has committed to a software patent non-aggression pledge. For details see [the original announcement](https://bsky.social/about/blog/10-01-2025-patent-pledge).

## P.S.

We ❤️ you and all of the ways you support us. Thank you for making Bluesky a great place!
