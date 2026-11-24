# UI lab results

Tool parts that I chose: Part 3: Robot Framework and Part 4: Cypress

## Part 0
- Step 0.1, Chrome version: Google Chrome 157.0.8103.79 (Official Build) (64-bit)
- Step 0.3, ChromeDriver version: ChromeDriver 157.0.8103.79 (6dc9bc3bdaea5c35ed6d41fceab2f7a3a2227c4f-refs/branch-heads/8103@{#1574})
- Step 0.4, locator of Pizza, Please!: `button[name="pizza-please"]` (or `name:pizza-please`)
- Step 0.4, locator of Excluded tools: `select` (or `css:select`)
- Step 0.4, locator of Must be vegetarian: `#default-checkbox` (or `id:default-checkbox`)
- Step 0.4, locator of Custom Pizza Name: `input#pizza-name` (or `css:input#pizza-name`)
- Step 0.5, pizza name, tool and calories of the run by hand: `Lab Pizza 20261124-095810`, Tool: `Scissors`, Calories per slice: `450`

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1 | The QuickPizza home page opened and the heading read `Looking to break out of your pizza routine?` | Pass |
| 2 | Turning on the `Advanced` switch revealed `Max Calories per Slice`, `Excluded tools`, `Must be vegetarian`, and `Custom Pizza Name` | Pass |
| 3 | Filled in `600`, selected `Knife`, ticked `Must be vegetarian`, and entered `Lab Pizza 20261124-095810`; all four fields displayed these values | Pass |
| 4 | The recommendation displayed `Name: Lab Pizza 20261124-095810`, `Tool: Scissors` (not `Knife`), and `Calories per slice: 450` (<= 600) | Pass |
| 5 | Clicking `Love it!` without logging in displayed `Please log in first.` | Pass |
| 6 | Opening `/admin` and signing in with `admin` / `admin` showed the `Latest pizza recommendations` heading and list | Pass |
| 7 | The admin list contained `Lab Pizza 20261124-095810 (tool=Scissors, ingredients_number=6) (newest)` matching the tool from Step 4 | Pass |

## Part 3: Robot Framework
- robot --version: `Robot Framework 7.5.1 (Python 3.13.9 on win32)`
- Step 3.2, failure message for the wrong title: `Title should have been 'Web Form' but was 'Web form'.`
- Step 3.8, run 1, result, elapsed time and pizza name: `PASS` (1 test, 1 passed, 0 failed), elapsed time `00:00:10` (`10.26s`), pizza name `Lab Pizza 20261124-100035`
- Step 3.8, run 2, result, elapsed time and pizza name: `PASS` (1 test, 1 passed, 0 failed), elapsed time `00:00:17` (`17.35s`), pizza name `Lab Pizza 20261124-100050`

## Part 4: Cypress
- node --version, npm --version and npx cypress version: Node.js `v24.14.1`, npm `11.11.0`, Cypress package version `16.2.1` (Cypress binary version `16.2.1`, Electron `41.7.0`, Bundled Node `24.15.0`)
- Step 4.3, failure message for the wrong title: `Timed out retrying after 10000ms: expected 'Web form' to equal 'Web Form'`
- Step 4.6, tool and calories in the answer of POST /api/pizza: `tool: "Pizza cutter"`, `calories: 400`
- Step 4.7, the line of the admin list that shows your pizza: `Lab Pizza 20261124-100307 (tool=Pizza cutter, ingredients_number=6) (newest)`
- Step 4.8, run 1, result, duration and pizza name: `All specs passed!` (2 passing), duration `00:13` (`pizza-scenario.cy.js` `00:10`, `web-form.cy.js` `00:02`), pizza name `Lab Pizza 20261124-100155`
- Step 4.8, run 2, result, duration and pizza name: `All specs passed!` (2 passing), duration `00:11` (`pizza-scenario.cy.js` `00:09`, `web-form.cy.js` `00:02`), pizza name `Lab Pizza 20261124-100307`
- Step 4.9 (optional), browser and result: Microsoft Edge 158 (`--browser edge`), both specs passed (`All specs passed!`, 2 passing in `00:13`)

## Part B
1. On the QuickPizza home page, querying `$$("#floating_filled").length` in the console returns `3` because all three number inputs (`Max Calories per Slice`, `Min Number of Toppings`, and `Max Number of Toppings`) share the non-unique `id="floating_filled"`. Consequently, an ID selector cannot distinguish the calories input from the other two boxes, so our tests locate the `<label>` containing `Max Calories per Slice` and step to its sibling `<input>`. To fix this on the page, each number input should be given a unique, descriptive `id` attribute such as `id="max-calories"`, `id="min-toppings"`, and `id="max-toppings"`. Having unique IDs helps a tester write short, fast, and resilient locators (`#max-calories`) that will not break if the DOM hierarchy or label wording around the input changes.
2. Comparing the tool displayed in the admin list (`(tool=Pizza cutter, ...)`) in Step 7 against the tool shown in the customer recommendation (`Tool: Pizza cutter`) in Step 4 is a **consistency oracle**, because it checks whether two different views of the same system agree with one another on a value generated at runtime. In our runs, the recommended tool varied randomly between `Scissors` and `Pizza cutter`, so a hardcoded expected tool could not be used. This consistency check would catch a fault where the backend saves a different tool to the database than the one returned to the customer UI, or where `/admin` renders stale or mismatched tool metadata for a pizza entry.
3. To verify that the recommended pizza is truly vegetarian, the test could inspect the response body of `POST /api/pizza` (which we intercepted with `cy.intercept('POST', '/api/pizza').as('pizza')` in Cypress Step 4.6) and assert `expect(response.body.vegetarian).to.equal(true)`, as well as checking that none of the items listed under `Ingredients:` on the page (`#recommendations ul li`) belong to a known list of meat ingredients (such as pepperoni, ham, sausage, prosciutto, or bacon). From the page or another source, the test would either need the JSON response from `/api/pizza` (which includes `"vegetarian": true`) or a complete reference list of non-vegetarian ingredients from the QuickPizza API (`/api/ingredients`) to validate the rendered ingredient list in the DOM.
4. Robot Framework with SeleniumLibrary requires explicit wait keywords (`Wait Until Element Is Visible`, `Wait Until Page Contains`, `Wait Until Element Contains` configured with `timeout=10s`) before acting on or asserting elements that SvelteKit renders asynchronously, whereas Cypress automatically retries every `cy.get()`, `cy.contains()`, and `.should()` assertion for up to `defaultCommandTimeout: 10000` ms without extra wait lines. Writing `QP-01` in Cypress was slightly faster because its chainable commands (`.clear().type('600').should('have.value', '600')`) and built-in `cy.intercept()` network inspection required less boilerplate than defining separate keywords and string-parsing steps in `quickpizza.resource`. For output readability, Robot Framework generates a self-contained, expandable HTML execution tree (`log.html` and `report.html`) with embedded screenshots at each step (`10.26s` and `17.35s`), while Cypress provides a clean terminal summary table (`00:13` and `00:11`), failure diffs, step screenshots, and full `.mp4` videos (`pizza-scenario.cy.js.mp4`) showing the Command Log side-by-side with the live browser viewport.
5. I would choose **Cypress** for the automated UI tests of Project 2. First, Cypress's automatic retry-and-wait mechanism on every DOM query and assertion eliminated timing flakiness on QuickPizza's asynchronous SvelteKit rendering without needing manual wait statements before every check. Second, Cypress records full `.mp4` videos (`cypress/videos/pizza-scenario.cy.js.mp4`) and lets tests inspect XHR/Fetch traffic with `cy.intercept('POST', '/api/pizza')`, which directly satisfies Project 2's evidence requirements and makes diagnosing both UI and API regressions straightforward.
6. (For Part 3: Robot Framework) If QuickPizza changed the locator of the calories box, I would update only the `${MAX CALORIES}` variable in the `*** Variables ***` section at the top of `robot/resources/quickpizza.resource` (currently `xpath://label[contains(.,'Max Calories')]/../input`). Because `pizza_scenario.robot` only calls high-level keywords like `Turn On Advanced` and `Fill In The Pizza Request    600    Knife    ${name}`, the test file itself never mentions `${MAX CALORIES}` or any XPath expression and does not need to be edited at all.
7. (For Part 4: Cypress) QuickPizza runs behind multiple backend server instances and uses a session cookie to pin a browser session to one server instance, and each server maintains its own in-memory list of recent pizza recommendations. Because Cypress clears all cookies and browser state before every `it(...)` test block for test isolation, splitting Steps 6 and 7 into a second `it(...)` block would discard the session cookie from Steps 1–5. When the second test opened `/admin`, the load balancer could route it to a different QuickPizza server instance where the newly created pizza (`Lab Pizza 20261124-100307`) does not exist, causing `cy.contains('li', pizzaName)` in Step 7 to time out after 10000ms and fail.
