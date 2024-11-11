---
date: 2024-10-24
---

> DETR将transformer引入到目标检测任务中

```py
# Query Interaction Module
new_track_query = last_detect_queries.filter()  # filter: Object Enterance
new_track_queries = last_track_queries.filter()  # filter: Object Exit

track_queries = new_track_queries + new_track_query

draw_box(frame)
```

