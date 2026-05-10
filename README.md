# To-Do App

A fully functional task management app built with HTML, CSS, Bootstrap 5, Font Awesome, and JavaScript. Every task is built entirely through JavaScript's DOM API — no hard-coded task markup exists in the HTML. Each task supports inline editing (toggle between read-only and editable via an Edit/Save button), strike-through completion on double-click, and permanent deletion via a trash icon button — all without any page reload or backend.

---

## What This Project Does

Typing a task into the input field and clicking the Add button (`+`) calls `addtask()`, which dynamically constructs a complete task row — a read-only text input, an Edit/Save button, and a Delete button — and appends it to the task container. Tasks live in the DOM for the session. Every interaction (edit, save, complete, delete) is handled by event listeners attached to each task's elements at the moment of creation.

---

## How the JavaScript Works — `Script.js`

**3 DOM elements selected on load:**
```js
var add           = document.getElementById("addToDo");
var input         = document.getElementById("inputfield");
var todocontainer = document.getElementById("todoconatiner");
```

---

### `addtask()` — Full DOM Construction Flow

Called via `onclick="addtask()"` on the Add button. Every task is built from scratch on each call.

**Step 1 — Empty input guard:**
```js
if (input.value === "") {
  alert("Please enter your Task!");
}
```
Strict equality check against empty string — a whitespace-only entry (`"   "`) would pass this check and be added as a task.

**Step 2 — Capture and clear input:**
```js
let item_value = input.value;
// ... build elements ...
input.value = "";    // clears the input field after task is added
```

**Step 3 — Task element tree built via `createElement`:**

The complete DOM structure created for each task:
```
div.item
├── div.content
│   └── input.text  [type="text", value=task, readonly]
└── div.actions
    ├── button.edit.btn.btn-success   "Edit"
    └── button.delete.btn.btn-danger.fa.fa-trash
```

- `div.item` — outer flex row for the whole task
- `div.content` — holds the task text input, takes `width: 90%`
- `input.text` — `type="text"`, pre-filled with the task text, `readonly` attribute set via `setAttribute('readonly', 'readonly')` — prevents typing until Edit is clicked
- `div.actions` — flex container for Edit and Delete buttons
- `edit_item` — Bootstrap `btn btn-success` (green), text "Edit"
- `delete_item` — Bootstrap `btn btn-danger` (red) + `fa fa-trash` class from Font Awesome — the trash icon is rendered by Font Awesome directly on the button element without a separate `<i>` tag

**Step 4 — Tree assembly order:**
```js
item_content.appendChild(input_item);   // text input → content div
item_action.appendChild(edit_item);      // Edit btn → actions div
item_action.appendChild(delete_item);    // Delete btn → actions div
item.appendChild(item_content);          // content → item
item.appendChild(item_action);           // actions → item
todocontainer.appendChild(item);         // item → main container
```

---

### Event Listeners — Attached Per Task at Creation Time

**Double-click to strike through (complete):**
```js
input_item.addEventListener('dblclick', function () {
  input_item.style.textDecoration = 'line-through';
});
```
A double-click on the task text applies `line-through` inline style — visually marking the task as done.

**Single click to un-strike (reactivate):**
```js
input_item.addEventListener('click', function () {
  input_item.style.textDecoration = 'none';
});
```
A single click removes the line-through, toggling the task back to active. Since `dblclick` also fires a `click` event first, the sequence on double-click is: `click` fires (removes strike) → `dblclick` fires (adds strike) — net result: strike-through is applied on double-click.

**Edit / Save toggle:**
```js
edit_item.addEventListener('click', (e) => {
  if (edit_item.innerText.toLowerCase() == "edit") {
    edit_item.innerText = "Save";
    input_item.removeAttribute("readonly");
    input_item.focus();
  } else {
    edit_item.innerText = "Edit";
    input_item.setAttribute('readonly', 'readonly');
  }
});
```
The button text itself is the state flag — no separate boolean variable. When the button reads "edit" (lowercased for comparison), clicking it removes `readonly` from the input, puts focus inside it for immediate typing, and changes the button label to "Save". Clicking "Save" restores `readonly` and resets the label to "Edit".

**Delete:**
```js
delete_item.addEventListener('click', (e) => {
  todocontainer.removeChild(item);
});
```
`removeChild(item)` removes the entire task row (`div.item`) — the outer container, both inner divs, the text input, and both buttons — in one call. The `item` variable is captured in the closure at task-creation time, so each delete button always references its own task.

---

## Styling — `Style.css`

**Indian tricolor background:**
```css
background: linear-gradient(0deg,
  green  0%,   green  25%,
  white  25%,  white  75%,
  orange 75%,  orange 100%
);
```
A vertical 3-band gradient replicating the Indian flag's saffron, white, and green colors (rendered top to bottom as orange → white → green). Hard color-stop values (`25%` and `75%`) create sharp, solid band edges with no feathering.

**Header (`#main-header`):** `background-color: navy` — a dark navy bar. `text-shadow: 2px 2px 2px #030202` on the title. The pencil icon (`fa fa-pencil`) is `float: right` inside the header row, placing it at the far right of the header.

**Container width:** `width: 50%` centered (`margin-top: 7em`). Shrinks responsively across 4 breakpoints:

| Breakpoint | Container width |
|---|---|
| Default (desktop) | `50%` |
| `max-width: 800px` | `70%` |
| `max-width: 650px` | `80%` |
| `max-width: 500px` | `99%` |
| `max-width: 375px` | `99%` + header font reduces from `3rem` to `2rem` |

**Task row (`.item`):** `display: flex; flex-direction: row; padding: 1rem; border-bottom: 4px solid rgb(249, 230, 146)` — a warm yellow bottom border separates each task. The `.content` div takes `width: 90%`, leaving `10%` for the actions.

**Task text input (`.text`):** `border: none; outline: none; border-color: transparent; font-size: 1.125rem; width: 100%` — completely invisible as an input field, blending into the card background. `cursor: context-menu` on hover signals the element is interactive without showing a text cursor unless in edit mode. `text-decoration: line-through` is applied inline by JavaScript when the task is completed.

**Action buttons (`.item .actions button`):** `margin-left: 0.5rem; font-weight: 700; font-size: 0.9rem; padding: 2px 6px` — compact sizing. Bootstrap's `btn-success` (green) and `btn-danger` (red) handle the button colors.

---

## Tech Stack

| Technology | Role |
|---|---|
| HTML5 | Static input field, Add button with `onclick`, empty `#todoconatiner` div |
| CSS3 | Indian tricolor gradient background, task row bottom border, container responsive sizing |
| Bootstrap 5.3.3 (CDN) | Card layout, `btn btn-success`, `btn btn-danger`, `input-group`, `form-control` classes |
| Font Awesome 6.7.2 (CDN) | `fa fa-pencil` in the header, `fa fa-trash` on delete buttons |
| JavaScript (Vanilla) | Full DOM construction per task, 4 event listeners per task (dblclick, click, edit toggle, delete) |

---

## Project Structure

```
To-Do-App/
├── Index.html      # Static shell — header, input group with Add button, empty #todoconatiner
├── Style.css       # Tricolor gradient body, navy header, task row border, 4 responsive breakpoints
└── Script.js       # addtask() — full DOM build per task, inline event listeners for complete/edit/delete
```

---

## How to Run

1. Clone the repository
   ```bash
   git clone https://github.com/tripathipawan/To-Do-App.git
   ```
2. Open `Index.html` directly in any modern browser — Bootstrap and Font Awesome load from CDN automatically. No server, no build step, nothing to install.

---

## Repository

[https://github.com/tripathipawan/To-Do-App](https://github.com/tripathipawan/To-Do-App)
