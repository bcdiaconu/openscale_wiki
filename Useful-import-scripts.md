## MiFit 'BODY' file --> openScale CSV file

by Martin1887 see https://github.com/oliexdev/openScale/issues/669

```python
#!/usr/bin/python

import argparse
import csv
import datetime

OPENSCALE_HEADER = '"biceps","bone","caliper1","caliper2","caliper3","calories","chest","comment","dateTime","fat","hip","lbm","muscle","neck","thigh","visceralFat","waist","water","weight"'

_MIFIT_BODY_HEADER = 'timestamp,weight,height,bmi,fatRate,bodyWaterRate,boneMass,metabolism,muscleRate,visceralFat,impedance'

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('mifit_BODY_csv_file_path')
    parser.add_argument('output_path')
    
    args = parser.parse_args()
    
    with open(args.mifit_BODY_csv_file_path, 'r') as inp:
        reader = csv.DictReader(inp)
        with open(args.output_path, 'w') as outp:
            writer = csv.DictWriter(outp, OPENSCALE_HEADER.replace('"', '').split(','))
            outp.write(f'{OPENSCALE_HEADER}\n')
            
            for line in reader:
                timestamp = int(line['timestamp'])
                output_date = datetime.datetime.fromtimestamp(timestamp).strftime('%Y-%m-%d %H:%M')
                weight = line['weight']
                writer.writerow({'dateTime': output_date, 'weight': weight})
                
```

## Libra CSV file --> openScale CSV file

by feclare see https://github.com/oliexdev/openScale/issues/28

```python
#!/usr/bin/env python
"""
Simple script to transform libra csv file to openscale format
"""
import sys
import csv
from dateutil.parser import parse

if len(sys.argv) != 2:
    print "Missing file to transform\n"
    sys.exit(1)

r = csv.reader(file(sys.argv[1]), delimiter=";")
writer = csv.writer(file("openScale_data_Libra.csv", "w"), delimiter=",")

for w in r:
    if len(w) == 0 or w[0].startswith("#"):
        continue
    time = w[0]
    weight = w[1]
    comment = w[5]
    d = parse(time)
    writer.writerow([d.strftime('%d.%m.%Y %H:%M'), weight, 0.0, 0.0, 0.0, 0.0, 0.0, comment])
```