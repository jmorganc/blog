Title: Zabbix kWh
Date: 4/22/2014, 4:26:18 PM
Category: Programming
Tags: python
Summary: Log energy usage to Zabbix.

This is a work in progress. It is currently published to test and choose syntax highlighting.

Process of getting info from state website.

Python processing.
```language-python
#!/usr/bin/python

import csv
import time

FILES = ('15m', 'daily', 'monthly')

for file in FILES:
    with open('IntervalMeterUsage_{file}.csv'.format(file=file), 'rb') as fh:
        meter_csv_obj = csv.reader(fh)
        meter_csv_obj.next()
        with open('IntervalMeterUsage_{file}.txt'.format(file=file), 'w') as txt_out:
            if file == '15m':
                for esiid, date, timestart, timeend, kwh, estact, congen in meter_csv_obj:
                    timestamp = time.mktime(time.strptime(' '.join((date, timestart)), '%Y-%m-%d %H:%M'))
                    txt_out.write('Bespin house.kwh.{file} {timestamp} {kwh}\n'.format(file=file, timestamp=timestamp, kwh=kwh))
            elif file == 'daily':
                for esiid, date, reading_start, reading_end, kwh in meter_csv_obj:
                    timestamp = time.mktime(time.strptime(date, '%m/%d/%Y'))
                    txt_out.write('Bespin house.kwh.{file} {timestamp} {kwh}\n'.format(file=file, timestamp=timestamp, kwh=kwh))
            elif file == 'monthly':
                for esiid, date_start, date, kwh, kwh_m, kwh_b, kva_m, kva_b in meter_csv_obj:
                    timestamp = time.mktime(time.strptime(date, '%m/%d/%Y'))
                    txt_out.write('Bespin house.kwh.{file} {timestamp} {kwh}\n'.format(file=file, timestamp=timestamp, kwh=kwh))
            else:
                sys.exit()
```

Zabbix sender.
```language-bash
zabbix_sender -z host -i IntervalMeterUsage_15m.txt -T -v
```
