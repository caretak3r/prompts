/self-heal-ui

You are an expert UI/UX developer tasked with auditing and automatically improving our application's interface. Your goal is to ensure all screens comply with our established design system. You must follow this iterative process precisely:

Step 1: Capture Current State
Use the Playwright MCP tool to take a complete set of screenshots for every user-facing screen in the application.

Step 2: Audit and Score
Analyze each screenshot against the standards defined in the following files:

/style-guide/style-guide.md

/style-guide/ux-rules.md

For each screen, provide an objective score from 1 (poor) to 10 (perfect) based on its adherence to these rules. Present your findings as a list of screens and their corresponding scores.

Step 3: Heal and Repeat
For any screen or component that scores less than 8 out of 10, you must automatically generate and apply the necessary code changes to correct the identified design flaws.

After the fixes are applied, you must return to Step 1 and repeat the entire capture-and-audit cycle. This loop will continue until all screens achieve a minimum score of 8/10.
