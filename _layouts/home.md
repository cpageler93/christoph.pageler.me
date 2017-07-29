---
layout: default
---

{% capture my_include %}{% include about.md %}{% endcapture %}
{{ my_include | markdownify }}
