# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): ...
- `size` (str): ...
- `max_price` (float): ...

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->

---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): ...
- `wardrobe` (dict): ...

**What it returns:**
<!-- Describe the return value -->

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (...): ...

**What it returns:**
<!-- Describe the return value -->

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->

---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | |
| suggest_outfit | Wardrobe is empty | |
| create_fit_card | Outfit input is missing or incomplete | |

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->

---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**

**Milestone 4 — Planning loop and state management:**

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->

The agent will look for items that match the user's criteria in the dataset.

The agent will take the description of what the user is asking for (e.g. t-shirt, pants, etc.), the size (if any), and the max price (if any) from the input prompt.

Then, it will call the `search_listings(description, size, max_price)` tool to get 3 items that match the user's criteria. The items will be retrieved from the static dataset (_listings.json_) and will be sorted by relevance.

The tool will not fail directly, but if it doesn't find any matches, it will return an empty list.
Also, in case the tool returns an empty list, it doesn't call the tool on **Step 2** and will provide an answer to the user to try another prompt in a different way. Something like:
- "_Hi! I couln't find an item that matches your criteria. What if you try with another piece and I'll help you figure out how to wear them!_"

The inputs are the following:
- **Description:** Keywords that describe or detail what the user is looking for (e.g. t-shirt, pants, etc.)
- **Size:** Size of the item. It can be empty or None to skip size filtering
- **Max price:** Maximum price the user will pay for the item. Inclusive. Can be empty to avoid filtering by price.

The output is the following:
- A list of 3 items that are the most relevant to the user's query
- **The most relevant item example:** _Faded Band Tee — $22, Depop, Good condition._

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->

After the agent gets the 3 most relevant items from the dataset, it will use the most relevant item and the current user's wardrobe to suggest a complete outfit.

From the tool on step 1, it uses the most relevant item. So, along with the user's current wardrobe (within `wardrobe_schema.json`), the agent will call the `suggest_outfit(new_item, user's wardrobe)` tool to generate a complete outfit suggestion (a string).

The tool will fail only if:
- The current user's wardrobe is empty, and it'll call the LLM with a prompt for general styling ideas (what kinds of items pair well with the item the user shared, what vibe it suits, etc.). 

The inputs are the following:
- **Most relevant item:** The most relevant item from the `search_listings(description, size, max_price)` tool on **Step 1**. Cannot be empty or null.
- **User's wardrobe:** The current user's wardrobe in a dictionary format with the list of items the user has in his/her wardrobe at home. It may be empty, so call an LLM for general styling ideas for the user.

The output is the following:
- A string with a complete outfit suggestion. Around 2 to 4 sentences long.
- **Example:** _Pair this with your wide-leg jeans and platform Docs for a classic 90s grunge look. Roll the sleeves once and tuck the front corner slightly for shape._


**Step 3:**
<!-- Continue until the full interaction is complete -->

After the agent gets the complete outfit suggestion from **Step 2** and the most relevant item from **Step 1**, it will send it to the LLM, using the `create_fit_card(suggested_outfit, new_item)` tool, along with a specific prompt, to generate a casual TikTok/Instagram ready-to-use caption (a string).

The tool won't fail directly, but if the outfit suggestion comes empty or missing, the agent will return a descriptive error message like:
- _Hi! It looks like I couldn't find an outfit suggestion for you. Can you try again with another item? I'll do my best this time!_

The inputs are the following:
- **Outfit suggestion:** The complete outfit suggestion from the `suggest_outfit(new_item, user's wardrobe)` tool on **Step 2**. Can be empty.
- **Most relevant item:** The most relevant item from the `search_listings(description, size, max_price)` tool on **Step 1**. Cannot be empty or null.

**Final output to user:**
<!-- What does the user actually see at the end? -->

_thrifted this faded band tee off depop for $22 and honestly it was made for my wide-legs 🖤 full look in my stories_

