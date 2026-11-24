# Automated UI Testing Lab (ITDS362)

This repository holds the automated UI testing lab using **Part 3: Robot Framework** and **Part 4: Cypress**.

## Tool Parts

### `robot/` (Part 3: Robot Framework)
Holds the Robot Framework test suites (`first.robot`, `web_form.robot`, and `pizza_scenario.robot`), the page object keyword resource file (`resources/quickpizza.resource`), and the saved reports, logs, and screenshots from two full headless runs (`results/run1/` and `results/run2/`).

Run the QP-01 scenario from the `robot` folder (with the virtual environment activated):
```powershell
robot -d results/run1 -v BROWSER:headlesschrome pizza_scenario.robot
```

### `cypress/` (Part 4: Cypress)
Holds the Cypress project configuration (`cypress.config.js`), end-to-end test specs (`cypress/e2e/web-form.cy.js` and `cypress/e2e/pizza-scenario.cy.js`), custom command definitions (`cypress/support/commands.js`), and the recorded videos and screenshots (`cypress/videos/` and `cypress/screenshots/`).

Run all Cypress specs in Chrome from the `cypress` folder:
```powershell
npx cypress run --browser chrome
```
