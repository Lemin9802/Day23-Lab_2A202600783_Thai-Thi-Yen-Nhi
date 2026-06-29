# Drift Detection Summary

The drift detection job compared reference vs current synthetic inference data.

Detected drift:
- prompt_length: PSI=3.461, KL=1.7982, KS=0.702, drift=yes
- response_quality: PSI=8.8486, KL=13.5011, KS=0.941, drift=yes

No drift:
- embedding_norm: PSI=0.0187, drift=no
- response_length: PSI=0.0162, drift=no

Conclusion:
The current dataset shows significant drift in prompt length and response quality. This means the input distribution and model quality signal changed enough to warrant investigation.
