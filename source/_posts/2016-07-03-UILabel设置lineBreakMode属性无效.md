---
title: "UILabel设置lineBreakMode属性无效"
date: 2016-07-03 18:09:30
categories: iOS Tips
tags:
  - UILabel
---

UILabel的lineBreakMode属性用于设置文字的截断方式，我在使用过程中，发现了一个问题，当我设置lineBreakMode为NSLineBreakByCharWrapping或NSLineBreakByWordWrapping时，有时会没有效果，它依然以NSLineBreakByClipping方式显示。

经过一番Google，原来，要设置NSLineBreakByCharWrapping或NSLineBreakByWordWrapping，UILabel的numberOfLines必须是0，否则就会无效。
