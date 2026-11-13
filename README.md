# omnipulse/github-sponsors

GitHub Sponsors for [omnisense/omnipulse](https://github.com/glitchr-omnisense/omnipulse), under any
PHP host: a user's or an organization's listing read through GitHub's GraphQL API - its page,
its goal, its tiers, the number of sponsors and the public ones.

Formerly `omnifund/github-sponsors` (omnifund merged into omnipulse on 2026-11-10: `Omnifund\GitHubSponsors\` is now
`Omnipulse\GitHubSponsors\`; this package replaces the former name).

## Install and a first call

```sh
composer require omnipulse/github-sponsors
```

No account for this first call: GitHub's GraphQL answer is recorded
(`Omnipulse\Testing\RecordedHttpClient` answers in place of the API), so nothing leaves the process.
A real call needs a GitHub token: GitHub answers no anonymous GraphQL call, but any token reads
public listings (a fine-grained token with no permission is enough).

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Omnipulse\GitHubSponsors\GitHubSponsorsFactory;
use Omnipulse\Testing\RecordedHttpClient;

// GitHub's GraphQL answer, recorded: nothing sent. HttpClient::create() (symfony/http-client) for a real call.
$http = RecordedHttpClient::routes([
    'POST /graphql' => '{"data": {"repositoryOwner": {"login": "octo-studio", "name": "Octo Studio", "url": "https://github.com/octo-studio", "hasSponsorsListing": true, "sponsors": {"totalCount": 3}, "sponsorsListing": {"name": "sponsors-octo-studio", "shortDescription": "Support Octo Studio\'s open source work", "url": "https://github.com/sponsors/octo-studio", "isPublic": true}}}}',
]);
$sponsors = (new GitHubSponsorsFactory($http))->create(['account' => 'octo-studio', 'token' => 'recorded']);

$profile = $sponsors->profile();
echo $profile->name, ': ', $profile->sponsors, " sponsors\n";   // Octo Studio: 3 sponsors
```

With your credentials, through the family's registry:

```php
use Omnipulse\GitHubSponsors\GitHubSponsorsFactory;
use Omnipulse\Registry;

$registry = new Registry([new GitHubSponsorsFactory($httpClient)], [
    'github' => ['factory' => 'github-sponsors', 'options' => ['account' => 'glitchr-studio', 'token' => $token]],
]);
$registry->sponsors('github')->sponsors();
```

Options: `account` (required), `token` (required: GitHub answers no anonymous GraphQL call; any
token reads public listings - a fine-grained token with no permission is enough), `endpoint`
(GitHub Enterprise: `https://HOST/api/graphql`), `user_agent`.

- **Profile**: the listing's name, short description and page, the account's picture, the
  number of sponsors (private ones included, as GitHub counts them), the active goal (an amount
  in dollars a month, or a number of sponsors, and its percentage). No income: GitHub shows it
  to the account alone.
- **Sponsors**: the public ones, 100 a page; a private sponsor is never listed. No tier per
  sponsor: GitHub shows it to the sponsor and the maintainer alone.
- **Tiers**: monthly or one-time, in US cents as GitHub prices them, custom amounts marked.

No library is wrapped: no maintained framework-free client covers GitHub Sponsors - a general
GitHub client (knplabs/github-api) would bring its own HTTP stack for two GraphQL queries.

## Terms

GitHub's [Terms of Service](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service),
the [GitHub Sponsors additional terms](https://docs.github.com/en/site-policy/github-terms/github-sponsors-additional-terms)
and the [GraphQL rate limits](https://docs.github.com/en/graphql/overview/rate-limits-and-node-limits-for-the-graphql-api)
(5,000 points an hour per token): cache what you read; the profile and tiers cost one query, each
page of sponsors one more.

## Usage policy

GitHub Sponsors's usage policy is declared as data in [`policy.json`](policy.json) (omnisense/contracts'
schema, named in `composer.json`'s `extra.omnisense.policy`), its terms read on
2026-11-10 (https://docs.github.com/en/graphql/overview/rate-limits-and-node-limits-for-the-graphql-api): 5,000 points an hour and 2,000 a minute per token (GraphQL, written as `points`, counted one per call: a lower bound; GitHub's `x-ratelimit-*` fields read with their epoch reset), 100 concurrent requests; rights unknown; a token and a User-Agent. omnipulse's registry enforces it - a call over a
limit is never made, a 429 and its `Retry-After` hold every next call
(`UsageLimitReachedException`) - and a site may tighten it, never loosen it
(omnisense/omnipulse's docs/policies.md).

## Verified, and not

- Verified on recorded answers (the shapes of live answers of 2026-10-24, the people invented),
  in the core's harness: the profile, an amount goal and a count goal, the tiers from the same
  answer (one query), paging and limits, an unknown account, an account without a listing, a bad
  token, the factory's required options; every request's endpoint, bearer token, User-Agent,
  query and variables; `BareTest`; PHPStan at the highest level.
- Verified live (2026-10-24, a token): glitchr-studio's listing; sindresorhus's tiers and two
  sponsors.
- Not verified live: an active goal (none at hand).

## Responsibility

This package is an independent, unofficial client: it is not affiliated with, endorsed by or
sponsored by GitHub, Inc. Its MIT licence covers this package's code only and grants no right to
the service or its data. Whoever installs and runs it is responsible for complying with GitHub's
terms, for obtaining what they require, and for any use made of what it returns; the authors
accept no liability for how it is used (see [LICENSE](LICENSE)).

## Licence

MIT.
