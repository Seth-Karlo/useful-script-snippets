# Awk snippets

## Converting a timestamp into seconds

### Why

To add up different times that can range from seconds to hours and compare them

### How

```TIMESTAMP='0:22:41'; echo $TIMESTAMP | awk -F: '{ print ($1 * 3600) + ($2 * 60) + $3 }'```
