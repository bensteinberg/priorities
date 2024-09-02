priorities
==========

This is a Vue application for prioritizing a list of options by
choosing one of each pair of options. It currently requires local
copies of [Vue](https://vuejs.org/) and [Pico
CSS](https://picocss.com/), but could equally well use CDNs.

Settings
--------

The settings are nested inside the accordion for documentation. This
may not be the best place for them.

The `bubble` setting makes unmade choices float to the top of the
list, so you don't have to scroll down the page to make all the
choices. It defaults to `false`.

The `peek` setting shows the scored and ordered list before all the
choices are made. It defaults to `false`.

Alternate version
-----------------

An experiment with modern tooling, in part in the interest of reducing
asset size, is in `vite/`. This uses the currently-recommended setup
for a Vue app, using Vite, the Composition API, components, and
TypeScript. It reduces the size of the application by half, from 236K
to 116K.
