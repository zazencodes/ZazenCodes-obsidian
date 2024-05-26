---
date: 2024-04-15
tags:
    - code-snip
hubs:
    - "[[pandas]]"
---

# Convert nested JSONL to CSV


The example below has nested pagespeed data, e.g.



```python
import argparse


parser = argparse.ArgumentParser()
parser.add_argument("--file", help="File to convert")
args = parser.parse_args()

if "field" in args.file:
    """
    {"date":"2024-04-14","device":"MOBILE","first_contentful_paint":{"numericValue":1499,"category":"FAST","percent_slow":0.0277,"percent_average":0.1323,"percent_fast":0.8399},"largest_contentful_paint":{"numericValue":1748,"category":"FAST","percent_slow":0.0256,"percent_average":0.072,"percent_fast":0.9024},"first_input_delay":{"numericValue":34,"category":"FAST","percent_slow":0.0065,"percent_average":0.0782,"percent_fast":0.9153},"cumulative_layout_shift":{"numericValue":0,"category":"FAST","percent_slow":0.006,"percent_average":0.0095,"percent_fast":0.9845},"page":{"tier":"Example","name_short":"Example","name":"Example","url":"https://www.example.com/","page_type":"Homepage","cms":"Wordpress (Monolithic)"}}
    """
    print("Converting field data")
    metric_fields = [
        "first_contentful_paint",
        "largest_contentful_paint",
        "cumulative_layout_shift",
        "first_input_delay",
    ]
    metric_items = [
        "numericValue",
        "category",
        "percent_slow",
        "percent_average",
        "percent_fast",
    ]

elif "lab" in args.file:
    """
    {"date":"2024-04-14","device":"MOBILE","performance_score":0.69,"first_contentful_paint":{"score":0.96,"numericValue":1452.5,"grade":"GOOD"},"speed_index":{"score":0.8,"numericValue":4060.8229256535196,"grade":"NEEDS IMPROVEMENT"},"largest_contentful_paint":{"score":1,"numericValue":1533.75,"grade":"GOOD"},"time_to_interactive":{"score":0.45,"numericValue":7738,"grade":"POOR"},"total_blocking_time":{"score":0.05,"numericValue":2404,"grade":"POOR"},"cumulative_layout_shift":{"score":1,"numericValue":0,"grade":"GOOD"},"page":{"tier":"Example","name_short":"Example","name":"Example","url":"https://www.example.com/","page_type":"Homepage","cms":"Headless Wordpress"}}
    """
    print("Convedrting lab data")
    metric_fields = [
        "first_contentful_paint",
        "speed_index",
        "largest_contentful_paint",
        "time_to_interactive",
        "total_blocking_time",
        "cumulative_layout_shift",
    ]
    metric_items = [
        "score",
        "numericValue",
        "grade",
    ]

else:
    raise ValueError

from pathlib import Path
import pandas as pd
import json

fp = Path(args.file)

data = []
missing_data = set()
for i, line in enumerate(fp.read_text().splitlines()):
    if line.strip():
        d = json.loads(line)

        # Unnest multiple fields
        for field in metric_fields:
            field_data = d.pop(field, {})
            if not field_data:
                missing_data.add(i)

            for field_item in metric_items:
                d[f"{field}_{field_item}"] = field_data.get(field_item)
                if not d[f"{field}_{field_item}"]:
                    missing_data.add(i)

        # Unnest one field
        page_data = d.pop("page")
        for page_item in ("name", "url", "page_type", "cms", "tier", "name_short"):
            d[f"page_{page_item}"] = page_data.get(page_item)
        data.append(d)
    else:
        print(f"Found empty line at row {i}")

print(
    f"Some form of missing data for {len(missing_data)} records {len(missing_data)/len(data)*100:.2f}%"
)

df = pd.DataFrame(data)

print(f"Loaded {len(df)} rows to df with dtypes:")
print(df.dtypes)
print(df)

fout = fp.with_suffix(".csv")
print(f"Writing to {fout}")
df.to_csv(fout, index=False)

```
