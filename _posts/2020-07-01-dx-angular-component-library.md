---
layout:     project
title:      Data Explorer Component Library
date:       2020-07-01T11:30:00-0700
image:      /images/projects/dxac-intro-dark-imac.png
summary:    The Data Explorer Component Library is the culmination of years of component creation and refinement in the Data Explorer product. My team extracted the reusable components from our Data Explorer codebase, added examples, detailed usage instructions, and UX guidance, and packaged them into a reusable component library available to all engineering teams at FireEye.
categories: work
tags:
- Angular
- D3
- RxJS
- TypeScript
- Web Components
permalink:  /work/projects/data-explorer-component-library
---

<p>The Data Explorer Component Library is the culmination of years of UI component creation and refinement in the Data Explorer product. My team extracted the reusable components from our Data Explorer codebase, added examples, detailed usage instructions, and UX guidance, and packaged them into a reusable component library available to all engineering teams at FireEye.</p>
<p class="clearfix"></p>
<figure class="captioned-img">
  <img src="/images/projects/dxac-dropdown-dark.png" data-action="zoom"/>
  <figcaption>The dropdown component example page</figcaption>
</figure>
<p class="clearfix"></p>
<p>I'm a strong proponent of shared component libraries, especially in enterprise environments where they help to standardize UX and reduce engineering effort. I was very proud of the UI components my team and I had developed for the X15 Enterprise UI and Data Explorer products so in late 2018, I directed a project to extract, generalize, and package these components for broader use. Christine Lam did amazing work leading the engineering effort to create the library and example application. I contributed to the library architecture, components, and UX, and implemented the build and deployment system to an npm repository and AWS S3.</p>
<figure class="captioned-img">
  <img src="/images/projects/dxac-themes.png" data-action="zoom"/>
  <figcaption>Example dark and light themes</figcaption>
</figure>
<p>The Data Explorer Component Library is built in Angular using the <a href="https://angular.io/guide/creating-libraries" target="_blank">Angular Library pattern</a> with Angular CLI and Webpack. The library exposes source and compiled styles and fully supports theming. A WCAG 2.0 compliant light and dark theme are included by default. Components can be imported individually so consumers only include what they use.</p>
<figure class="captioned-img">
  <img src="/images/projects/dxac-donut-dark.png" data-action="zoom"/>
  <figcaption>The pie chart component example</figcaption>
</figure>
<p>In addition to the styling system and components, we included an example application with detailed instructions for combining components into more complex patterns like multi-step modals and stateful side navigation. The example application details the interface for each component and provides sample data and working examples to experiment with.</p>
<p class="clearfix"></p>

<h3>Credits</h3>
<ul class="credits">
  <li>UI/UX:
    Christine Lam,
    Phil Puleo,
    <a href="https://www.sebastiantoomey.com" target="_blank">Sebastian Toomey</a>
  </li>
  <li>Engineering:
    <a href="https://www.linkedin.com/in/benstone10" target="_blank">Ben Stone</a>,
    <a href="https://www.linkedin.com/in/bharat-jain-40546a36" target="_blank">Bharat Jain</a>,
    Christine Lam,
    <a href="https://www.linkedin.com/in/narcisse-noumegni-911a1a25" target="_blank">Narcisse Noumegni</a>,
    Phil Puleo
  </li>
</ul>