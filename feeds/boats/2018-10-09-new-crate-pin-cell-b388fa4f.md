---
title: 'New crate: pin-cell'
url: https://without.boats/blog/pin-cell/
published: "2018-10-09T00:00:00Z"
feed: boats
guid: https://without.boats/blog/pin-cell/
---

# New crate: pin-cell

Today I realized a new crate called pin-cell. This crate contains a type called PinCell, which is a kind of cell that is similar to RefCell, but only can allow pinned mutable references into its interior. Right now, the crate is nightly only and no-std compatible.
How is the API of PinCell different from RefCell? When you call borrow\_mut on a RefCell, you get a type back that implements DerefMut, allowing you to mutate the interior value.
