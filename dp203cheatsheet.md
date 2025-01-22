# My Azure DP-203 Cheat Sheet
Things you need to know well for the DP-203 exam.

# Windowing functions
Just a great topic that I imagine writers of exam questions love. Here is most of what you need to know.

| Type | Contiguous? | Overlapping? | Window size | Window starts when.. |
| ---- | ----------- | ------------ | ----------- | -------------------- |
| Tumbling | Yes | No | Fixed | At fixed time |
| Hopping | No | Yes | Fixed | Fixed |
| Sliding | No | Yes | Fixed? | When event enters/exits |
| Session | No | No | Variable | When first event occurs |
| Snapshot | No | No window | N/A | N/A |

