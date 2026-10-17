# Safety and limitations

The notebooks use best-effort prompt screening and image classification. These controls are not guarantees.

- Keyword screening is not semantic understanding.
- Image classifiers can produce false positives and false negatives.
- Video checks based on sampled frames can miss content between samples.
- Classifier errors, unreadable media, and incomplete scans should fail closed.
- Verify all ComfyUI output locations and downloaded copies before claiming flagged content is removed.
- Do not describe a run as fully safe merely because its classifier score is below a threshold.

Record the model name/revision, threshold, frames checked, flagged status, and cleanup result for each published test.
