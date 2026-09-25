# Vanilla JS DOM Query & Manipulation Methods (Q&A notes)

Plain JavaScript ("Vanilla JS") DOM methods that cover most of what jQuery used to be reached for.

## Contents
- [Selecting elements](#selecting-elements)
- [Classes](#classes)
- [Content and attributes](#content-and-attributes)
- [Creating and modifying the tree](#creating-and-modifying-the-tree)
- [Events](#events)

---

## Selecting elements

```javascript
document.querySelector("h1");                 // first match
document.querySelectorAll(".item");            // NodeList of all matches
document.getElementById("uniqueId");            // by id
document.getElementsByClassName("item");        // live HTMLCollection by class
document.getElementsByTagName("p");             // live HTMLCollection by tag
```
`getElementsBy*` results are live collections — convert with `Array.from(...)` to safely `.forEach()` over them, since `querySelectorAll` already returns a static NodeList that supports `.forEach()` directly.

## Classes

```javascript
const el = document.querySelector("p");
el.classList.add("highlight");
el.classList.remove("highlight");
el.classList.toggle("highlight");
el.classList.contains("highlight");   // true/false
```

## Content and attributes

```javascript
const div = document.querySelector("div");
div.innerHTML;                 // get/set HTML content
div.innerHTML = "<p>New content</p>";
div.textContent;               // get/set text content only

const link = document.querySelector("a");
link.setAttribute("href", "https://example.com");
link.getAttribute("href");
link.removeAttribute("alt");
link.hasAttribute("alt");      // false after removal

const button = document.querySelector("button");
button.style.backgroundColor = "blue";   // inline style
```

## Creating and modifying the tree

```javascript
const list = document.querySelector("ul");
list.insertAdjacentHTML("beforeend", "<li>New item</li>");

const div = document.querySelector("div");
const p = document.createElement("p");
p.textContent = "Appended paragraph";
div.appendChild(p);

const firstItem = list.querySelector("li");
list.removeChild(firstItem);

const button = document.querySelector("button");
const clone = button.cloneNode(true);   // true = deep clone (include children)
document.body.appendChild(clone);

const newItem = document.createElement("li");
newItem.textContent = "Replaced item";
list.replaceChild(newItem, list.firstElementChild);

const textNode = document.createTextNode("Hello World");
document.body.appendChild(textNode);
```

## Events

```javascript
const button = document.querySelector("button");

function handleClick() { console.log("Button clicked"); }

button.addEventListener("click", handleClick);
button.removeEventListener("click", handleClick);   // must pass the same function reference

button.dispatchEvent(new Event("click"));            // manually fire the event
```
- To remove a listener later, the handler must be a named function (or a stored reference) — an inline anonymous function passed to `addEventListener` can't be removed with `removeEventListener`.

# 색인과 출처
- 개인 Q&A 학습 노트 — Vanilla JS DOM 조작/조회 메서드 레퍼런스
