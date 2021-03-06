Title: The Juju charm store

# The Juju Charm Store

Juju includes a charm store where charms and bundles can be uploaded,
released (published), and optionally shared with other users.

The charm store is broken down into two main sections: Recommended and
Community. Recommended charms have been vetted and reviewed by a Juju
Charmer and all updates to the charm are also vetted prior to landing.
Community charms have been shared by members of the community, but were
not submitted by and have not been vetted by Juju Charmers.

  - [Charm development information](developer-getting-started.html)
  - [Bundle creation information](charms-bundles.html)

!!! Note: 
    In order to interact with the charm store you will need the latest
    [charm command](tools-charm-tools.html), an
    [Ubuntu SSO account](https://login.ubuntu.com/+login), and you must have
    logged in to [Launchpad](https://launchpad.net/+login) at least once.

## Log in to the store

It is required that you first log in to the
[Charmstore](https://jujucharms.com) before you attempt to log in via `charm
login`. After logging in via your browser, you can use the `charm`
command line tool.

Most charm commands require authentication in order to operate. You can
log in or log out of the store at any time by issuing `charm login` or
`charm logout` respectively. During login you will be prompted for the
following information:

 - `Username` - typically the email address used to access Ubuntu SSO
 - `Password` - Ubuntu SSO password
 - `Two-factor auth` - If enabled, enter two-factor authentication (2FA)
information. Otherwise <kbd>Return</kbd> for None

Also note, if your group memberships change on Launchpad, you'll need to
log out and log in via a browser to the
[Charmstore](https://jujucharms.com) before those membership changes are
recognized.

## Entities explained

When a charm or bundle, referred to as entity from this point forward, is
pushed for the first time to the store, the entity is named version 0 in
the unreleased (unpublished) channel. Every revision of an entity lives in
the unreleased channel. Subsequent pushes of different content to the store
will automatically increment this number. So if an entity is changed and
pushed 4 times, the revision history would look like this:

```
entity: 0---1---2---3
```

Channels group entity revisions into named streams. There are currently four
released channels: edge, beta, candidate, and stable. Each channel also
tracks history of revisions. When a revision is released to a channel, that
channel pointer is changed and the history for the channel is updated.

Building on the previous example, when a user released revision 2 to the
stable channel, the history would look like this:

```
                  stable (2)
                 /
entity: 0---1---2---3
```

If, then, revision 3 is released to the edge channel the history
would look like this:

```
                  stable (2)
                 /
entity: 0---1---2---3
                     \
                      edge (3)
```

During this time, more revisions can be pushed to the default, unreleased
channel. This represents general development iterations. As iterations are
pushed during development, the stable and other channels are not
updated.


```
                  stable (2)
                 /
entity: 0---1---2---3---4---5---6
                     \
                      edge (3)
```

The author can, at any time, release a revision to a channel. Revisions
can also exist in the same channel at the same time. For example, the
author chooses to release revision 3 to the stable channel without
updating the edge channel:

```
                      stable (3, 2)
                     /
entity: 0---1---2---3---4---5---6
                     \
                      edge (3)
```

In doing so, the stable channel is updated to point to revision 3 and
revision 3 is added to the channel history. The author can continue to
push and release. Since revisions are a constant stream there are
scenarios where the stable channel may be pointed to a higher revision
even though edge revision is actually newer.

In the following example revision 8 is edge, a bug is found in the
latest stable revision (5) so a hot fix is applied and pushed as 9.
That revision is then released to the stable channel, like this:

```
                                              stable (9, 5, 3, 2)
                                             /
entity: 0---1---2---3---4---5---6---7---8---9
                                         \
                                          edge (8, 3)
```

While authors can release older versions to the channels it is not
encouraged. For example, an author could mistakenly release revision 10 to
the stable channel and not edge, then the author could re-release
revision 9 to stable:

```
                                              stable (9, 10, 9, 5, 3, 2)
                                             /
entity: 0---1---2---3---4---5---6---7---8---9---10
                                         \
                                          edge (8, 3)
```

If a user managed to deploy that first mistaken revision during the time
it was available, they would later be notified of an "upgrade" by Juju
which will effectively downgrade the charm back to revision 9.

## Pushing to the store

After building a charm or bundle, navigate to it's directory on disk and
push it to the store.

```
cd src/charms/foobar
charm push .
```

The `charm push` command will return the full ID for the pushed item.
Since this is the first time foobar was pushed, the output of the command
is:

```
cs:~USER/foobar-0
```

If a charm or bundle id is not provided, they will default to
`cs:~USER/NAME` where `USER` is the `User` from the output of
`charm whoami` and `NAME` is the metadata.yaml `name` for charms and
directory basename for bundles.

To define a series or different bundle name an id can be provided during
push. The following is a set of different support permutations given the
following `charm whoami` output:

```
User: kirk
Group membership: charm-examples
```

User `kirk` can perform the following operations:

```
charm push .
charm push . charm-name
charm push . bundle/bundle-name
charm push . ~charm-examples/charm-name
charm push . cs:~charm-examples/charm-name
```

Push will always increment the charm version in the unreleased channel.

## Releasing to channels

The charm store supports four released channels: edge, beta, candidate, and
stable. Revisions are associated with channels by using the release charm
command. Release is executed against an existing revision and places that
revision as the channel pointer.

Given the following example:

```
charm push . foo
cs:~kirk/foo-9
```

The author could release foo-9 to either the stable or edge channel
as follows, showing the commands for stable and edge respectively:

```
charm release cs:~kirk/foo-9
charm release cs:~kirk/foo-9 --channel edge
```

After running both commands, revision 9 exists in both the stable channel
and the edge channel.

## Sharing charms and bundles

Sharing is independent of promulgating (making public as the recommended charm)
to the Charm Store.

All channels have read and write ACLs. By default, only the owner of the
entity exists in these ACLs.

To update the ACL for an entity you must grant users an ACL to the channel
you want them to access. By default, if you do not supply an entity when
granting access, the recipient will receive only read access to the
stable channel, as in this example where we grant james access to the
stable channel of the cs:~kirk/foo entity.

```
charm grant cs:~kirk/foo james
```

If, instead you wanted to give write access to the edge channel to lars, you
would issue the following command:

```
charm grant cs:~kirk/foo --channel edge --acl write lars
```

Finally, to make the entity available for all to consume, there is a
special `everyone` user you can use to make an entity available to the
general public.

```
charm grant cs:~kirk/foo everyone
```

## Promulgate your charm

When you have released your charm (or bundle) and you want to make it available
to others as the recommended charm you will need to make a *promulgation
request*. This is informally done via the "Charms and Charming" category on the
[Juju Discourse forum][juju-discourse-forum-charms].

The '#juju' IRC channel on Freenode and the above Discourse forum remain
excellent resources for questions and comments regarding charm development and
charm promulgation.

### Promulgation notes

 - The [Charm promulgation][charm-promulgation] page contains information on what
   happens once the request is made.
 - It is the responsibility of the charm author (and maintainer) to test
   their charm to ensure it is of good quality and is secure.
 - Promulgation to the top level namespace of the Charm Store does not imply
   an endorsement by Canonical.
 - Charm authors are encouraged to use their personal or group namespace.

<!-- LINKS -->

[juju-discourse-forum-charms]: https://discourse.jujucharms.com/c/charms
[charm-promulgation]: ./charm-promulgation.md
