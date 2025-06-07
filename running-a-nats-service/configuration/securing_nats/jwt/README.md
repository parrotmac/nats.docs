# Decentralized JWT Authentication/Authorization

Along with other authentication mechanisms, configurations for identifying a user and [Account](../accounts.md) are in the server configuration file. JWT authentication leverages [JSON Web Tokens \(JWT\)](https://jwt.io/) to describe the various entities supported. When a client connects, servers verify the authenticity of the request using [NKeys](../auth_intro/nkey_auth.md), download account information, and validate the trust chain. Users are not directly tracked by the server, but rather verified as and belonging to an [Account](../accounts.md). This enables the management of users without requiring server configuration updates.

Effectively, JWTs improve accounts and provide for a **distributed configuration paradigm**. Previously each user \(or client\) needed to be known and authorized a priori in the server’s configuration, requiring an administrator to modify and update server configurations. With JWTs, these chores are eliminated. User creation and verification can even be performed by different entities altogether.

> Note: This scheme improves [accounts](../accounts.md). Functionalities like [isolation](../accounts.md) or defining [exports/imports](../accounts.md#exporting-and-importing) between accounts remain! It moves configuration of accounts, exports/imports or users and their permissions away from the server into several trusted [JSON Web Token \(JWT\)](https://jwt.io/) that are managed separately, therefore removing the need to configure these entities in each and every server. It furthermore adds functionalities like expiration and revocation for decentralized account management.

## JSON Web Tokens

[JSON Web Tokens \(JWT\)](https://jwt.io/) are an open and industry standard [RFC7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties.

Claims are a fancy way of asserting information on a _subject_. In this context, a _subject_ is the entity being described -- not a messaging subject. JWT claims are typically digitally signed and verified.

NATS further restricts JWTs by requiring that JWTs be:

* Digitally signed _always_ and only using [Ed25519](https://ed25519.cr.yp.to/). 
* NATS requires that all _Issuer_ \(`iss`\) and _Subject_ \(`sub`\) fields in a JWT claim must be a public [NKEY](../auth_intro/nkey_auth.md). 
* The _Issuer_ and _Subject_ must match specific roles depending on the claim [NKeys](https://github.com/nats-io/nkeys).

### NKey Roles

[NKeys](../auth_intro/nkey_auth.md) Roles are:

* Operators
* Accounts
* Users

Roles are hierarchical and form a chain of trust. Operators issue Accounts which in turn issue Users. Servers trust specific Operators. If an Account is issued by an Operator that is trusted, Account Users are trusted.

## The Authentication Process

When a _User_ connects to a server, it presents a JWT issued by its _Account_. The user proves its identity by signing a server-issued cryptographic challenge with its private key. The signature verification validates that the signature is attributable to the user's public key. Next, the server retrieves the associated account JWT that issued the user. It verifies the _User_ issuer matches the referenced account. Finally, the server checks that a trusted _Operator_ - one the server is configured with - issued the _Account_, completing the trust chain verification.

## The Authorization Process

From an authorization point of view, the account provides information on messaging subjects that are imported from other accounts \(including any ancillary related authorization\) as well as messaging subjects exported to other accounts. Accounts can also bear limits, such as the maximum number of connections they may have. A user JWT can express restrictions on the messaging subjects to which it can publish or subscribe.

When a new user is added to an account, the account configuration need not change, as each user can and should have its own user JWT that can be verified by simply resolving its parent account.

## JWTs and Privacy

One crucial detail to keep in mind is that while in other systems JWTs are typically used as sessions or proof of authentication, NATS JWTs are only used as configuration describing:

* the public ID of the entity
* the public ID of the entity that issued it
* capabilities of the entity

Authentication is a public key cryptographic process — a client signs a nonce proving identity while the trust chain and configuration provides the authorization.

The server is never aware of the contents of private keys, but can verify that a signer or issuer indeed matches a specified or known public key.

Lastly, all NATS JWTs \(Operators, Accounts, Users and others\) are expected to be signed using the [Ed25519](https://ed25519.cr.yp.to/) algorithm. If they are not, they are rejected by the system.

## Decentralized Authentication and Authorization - Configuration and `nsc`

There is very little to configure on the nats-server to enable operator JWT security. Once the servers have been initially configured, the authentication and authorization tasks are typically done by using the `nsc` administration tool locally and synchronizing with the account resolvers built into the nats-server.

Configuration is broken up into separate steps. Depending on organizational needs these are performed by the same or different entities.

Practically, JWT configuration is done using the [`nsc` tool](../../../../using-nats/nats-tools/nsc/README.md). It can be set up to issue [NKeys](../auth_intro/nkey_auth.md) and corresponding JWTs for all [nkey roles](#nkey-roles): Operator, Account, and User \([Example usage](../../../../using-nats/nats-tools/nsc/basics.md#creating-an-operator-account-and-user)\). Despite Account and User creation not happening in server configuration, this model provides a centralized authentication and authorization setup.

Provided institutional trust, it is also possible to use `nsc` to import account or user public [NKeys](../auth_intro/nkey_auth.md) and issue corresponding JWTs. This way, one entity can issue account JWTs -- signed by an Operator -- and a separate entity can issue JWTs for Users. In this scenario, neither entity has to be aware of the other's private Nkey. This scheme not only allows users to be configured some place other than servers, but also enables configuration and issuance by different organizations altogether. For example, administrators of a NATS installation may control Operator JWTs and issue Account JWTs to individual production or development teams who, in turn, manage their own Users. This is a fully decentralized authorization setup!

With an Operator JWT in place, a server needs to be configured to trust it by specifying `operator`. Furthermore the server needs a way to obtain account JWTs. This done by either defaulting to the resolver specified in the operator jwt or by manually specifying the [resolver](resolver.md). Depending on your configuration an [account server](../../../../using-nats/nats-tools/nsc/basics.md#account-server-configuration) may be required.

> It is possible to [mix](jwt_nkey_auth.md) JWT and [NKEY](../auth_intro/nkey_auth.md)/[Account](../accounts.md)-based Authentication/Authorization.

# Managing JWT authentication

More information is available in the [In Depth Guide](../../../../running-a-nats-service/nats_admin/jwt.md).
