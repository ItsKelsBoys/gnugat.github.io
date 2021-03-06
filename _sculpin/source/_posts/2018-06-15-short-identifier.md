---
layout: post
title: Short Identifier
---

Sometimes resources can be identified by one of their attributes
(a name, title, slug) and sometimes they can't (no name, or confidential name).

In the later case, an ID needs to be artificially crafted.

Two of the popular strategies is to use either an auto incremental one
or a universally unique one, however when it comes to share them publicly,
both strategies present some drawbacks

## Auto Incremental IDs

Resources are given a serial number, starting from 1
and increased by 1 for every new entry added.

This means that the ID of the last resource needs to be known
in order to create a new one, so that's usually done by the database itself.

### Inconvenients:

* the ID isn't known until the actual addition is done
* the IDs are guessable
* the ID is only unique to this resource in this database

## Universally Unique IDs

Resources are attributed with a 36 characters long hexadecimal string,
which is computed by an algorithm.

A couple of Universally Unique ID (UUID) strategies are available,
the main ones being:

* v1: generation done using a timestamp and the MAC address of the computer
* v4: generation done using a secure random number generator

### Inconvenients:

* the ID is quite lengthy
* the ID is made of hexadecimal numbers
* the ID is hard to read and remember
  (because of the two previous bullet points)

## Short IDs

Since UUIDs are 36 characters long,
and auto incremental IDs are incrementally longer,
there might be a need for a shorter ID.

The requirements for a short ID are usually:

* be readable (e.g. 6 characters long)
* be automatically generated by an algorithm that is:
  * collision free (still needs to be unique)
  * fast (i.e. no possibility of infinite loop)
  * cryptologically secure (they shouldn't be predictable)
* be generated by the application as opposed to the database
  (allows for asynchronous environments)

### Hashing and collisions

A possible compromise is to keep the IDs
as they currently are in the system for internal operations,
and provide a hash of it for public operations,
with the hope that the hash is shorter and more readable than the ID.

As of 2018, the best hashing algorithm is
[SHA-256](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms),
as it generates 256 cryptologically secure bits
and hasn't been found to be vulnerable to
[collision attacks](https://en.wikipedia.org/wiki/Collision_attack)
_yet_.

The drawbacks of hashes are the following:

* a stronger algorithm needs to be adopted as collision attacks are found
  (as it happened for MD5 in 1996, and SHA1 in 2017)
* the stronger the algorithm, the longer the output bits
* they aren't collision free

Following the
[Birthday Paradox](https://en.wikipedia.org/wiki/Birthday_problem),
we can calculate when the first collision will happen:
after outputting `2 ** (n / 2)` hashes
(with **n** being the number of output bits).

For SHA-256, that's 340 undecillion
(340 282 366 920 938 463 463 374 607 431 768 211 456)
hashes before a collision happens.

> *References*:
>
> * [URL shortening hashes in practice](https://blog.codinghorror.com/url-shortening-hashes-in-practice/)
> * [How big is 2 ** 128](http://bugcharmer.blogspot.co.uk/2012/06/how-big-is-2128.html)

### Shortening hashes

The Git project uses hexadecimal representation of SHA-1 hashes
to identify each commits.

Since these hashes are quite lengthy (160 bits, 40 hexadecimal characters),
Git allows to use the first few characters to be able to identify it
(at first it was the first 7, then it's been changed to the first 12
and finally it's been changed to dynamically increment).

The number of hashes a subset covers can be calculated as follow: `16 ** d`
(with **d** being the number of first hexadecimal characters selected).

Using the Birthday Problem formula,
we can estimate that the first collision might happen after
`2 ** (n / 2)` short hashes have been used
(with **n** being the number of first bits, which is `d * 4`,
with **d** being the number of first hexadecimal characters).

Here's a handy list:

* 6 first characters: covers    16 777 216 hashes, but first collision happens after  4 096 hashes generated
* 7 first charecters: covers   268 435 456 hashes, but first collision happens after 16 384 hashes generated
* 8 first characters: covers 4 294 967 296 hashes, but first collision happens after 65 536 hashes generated

> *References*:
>
> * [Smallest SHA-1 prefix before collision?](https://www.quora.com/Cryptography-What-is-the-smallest-prefix-length-of-an-SHA1-hash-that-would-guarantee-uniqueness-in-a-reasonable-object-space)
> * [Git hashes](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection#Single-Revisions)
> * [Git bumping default from 7 characters to configurable one](https://github.com/git/git/commit/dce96489162b05ae3463741f7f0365ff56f0de36)
> * [Git introducing a way to automatically guess how many characters should be used](https://github.com/git/git/commit/e6c587c733b4634030b353f4024794b08bc86892)

### Alphabetical representation (to avoid)

At the lowest level, IDs are represented as binary numbers.

To be more human friendly, they can then be converted to a different base:

* base 10 for decimal integer (like IDs)
* base 16 for hexadecimal number (like hashes)

We can even use a custom base,
such as base 62 which would use all 26 English alphabet characters,
both lower case and upper case, and all 10 digits.

The bigger the base, the shorter the same number will be represented,
so a way to shorten an ID (and that includes a way to shorten a hash)
is to represent it in a different base.

While using this approach might seem like a good idea at first
(human readable IDs that look like words), they actually bring more trouble
(chances to generate bad words in many languages).

> *References*:
>
> * [The automated curse generator](https://thedailywtf.com/articles/The-Automated-Curse-Generator)

## Conclusion

Regardless of the type of internal ID used (auto incremental, UUID, or other),
it could still be benificial to generate from them a short public ID.

A way to do so is to create a SHA-256 hash out of the ID,
and then use its first few characters.

The precise number of character to use can be incremental,
it should be bumped up when the number of generated hashes becomes close
to the estimated first collision.

> *See also*:
>
> * [Convert a positive integer into a short ID](https://hashids.org/)
> * [Generate a short ID](https://github.com/ai/nanoid) (javascript only)
> * [Generate a short ID, sortable by creation](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c)
