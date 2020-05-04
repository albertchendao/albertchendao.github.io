---
layout: article
titles:
  en      : &EN       About
  en-GB   : *EN
  en-US   : *EN
  en-CA   : *EN
  en-AU   : *EN
  zh-Hans : &ZH_HANS  关于
  zh      : *ZH_HANS
  zh-CN   : *ZH_HANS
  zh-SG   : *ZH_HANS
  zh-Hant : &ZH_HANT  關於
  zh-TW   : *ZH_HANT
  zh-HK   : *ZH_HANT
---

我是 Albert Chen。

信仰「编程即是艺术」。

{% for category in site.data.skills %}
### {{ category.name }}

{% for keyword in category.keywords %}
- {{ keyword }}
{% endfor %}

{% endfor %}