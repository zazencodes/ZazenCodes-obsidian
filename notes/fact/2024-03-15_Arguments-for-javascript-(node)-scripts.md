---
date: 2024-03-15
tags:
    - fact
hubs:
    - "[[javascript]]"
---

# Arguments for javascript (node) scripts


Here's an example

```javascript
function parseArgs() {
  const args = process.argv.slice(2);

  const lmodDaysIdx = args.indexOf('--lastmod-days-window');
  if (lmodDaysIdx === -1 || args.length <= lmodDaysIdx + 1) {
    console.error('Please provide --last-mod-days-window argument to set maximum upload window.');
    process.exit(1);
  }
  const lmodDays = parseInt(args[lmodDaysIdx + 1]);

  const fileFilterIdx = args.indexOf('--file-filter');
  const fileFilter = (fileFilterIdx === -1 || args.length <= fileFilterIdx + 1) ? null : args[fileFilterIdx + 1]

  const updateExisting = process.argv.indexOf('--update-existing') > -1;
  const dryRun = process.argv.indexOf('--dry-run') > -1;

  const res = {
    lmodDays,
    fileFilter,
    updateExisting,
    dryRun,
  }

  console.log(JSON.stringify(res));
  return res;

}


```
