---
title: "RainyCyan's Blog"
---

👋 欢迎来到 RainyCyan 的博客

这是我的技术学习笔记，主要关注**数据库、存储引擎、向量检索、算法与数据结构**等领域。

<div id="daily-quote" class="daily-quote"></div>

<script>
const quotes = [
  "\"Talk is cheap. Show me the code.\" — Linus Torvalds",
  "\"Any fool can write code that a computer can understand. Good programmers write code that humans can understand.\" — Martin Fowler",
  "\"First, solve the problem. Then, write the code.\" — John Johnson",
  "\"It's not at all important to get it right the first time. It's vitally important to get it right the last time.\" — Andrew Hunt",
  "\"The best programs are written so that computing machines can perform them quickly and so that human beings can understand them clearly.\" — Donald Knuth",
  "\"Simplicity is the soul of efficiency.\" — Austin Freeman",
  "\"Make it work, make it right, make it fast.\" — Kent Beck",
  "\"Programs must be written for people to read, and only incidentally for machines to execute.\" — Harold Abelson",
  "\"The most disastrous thing that you can ever learn is your first programming language.\" — Alan Kay",
  "\"Measuring programming progress by lines of code is like measuring aircraft building progress by weight.\" — Bill Gates"
];
// 基于日期选择名言，每天固定一条
const today = new Date();
const dayOfYear = Math.floor((today - new Date(today.getFullYear(), 0, 0)) / 86400000);
const idx = dayOfYear % quotes.length;
document.getElementById('daily-quote').textContent = '> ' + quotes[idx];
</script>
