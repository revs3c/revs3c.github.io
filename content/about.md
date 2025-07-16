---
title: About
type: about
---

This is the about page.

{{< cards >}}
  {{< card link="../callout" title="Card with default tag color" tag="tag text" >}}
  {{< card link="../callout" title="Card with default red tag" tag="tag text" tagType="error" >}}
  {{< card link="../callout" title="Card with blue tag" tag="tag text" tagType="info" >}}
  {{< card link="../callout" title="Card with yellow tag" tag="tag text" tagType="warning" >}}
{{< /cards >}}

{{< cards >}}
  {{< card link="/" title="Image Card" image="https://source.unsplash.com/featured/800x600?landscape" subtitle="Unsplash Landscape" >}}
  {{< card link="/" title="Local Image" image="/images/card-image-unprocessed.jpg" subtitle="Raw image under static directory." >}}
  {{< card link="/blogs/windows" title="Local Image" image="/images/OIP.jpeg" subtitle="Image under assets directory, processed by Hugo." >}}
{{< /cards >}}
