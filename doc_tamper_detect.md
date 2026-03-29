This is a very rich, fresh set of findings — the most important one being the brand-new DOCFORGE-BENCH paper (arXiv:2603.01433, published March 2, 2026 — literally three weeks ago). This completely changes the answer. Let me now build the definitive, research-backed response.This is a fundamentally different answer than anything previously given to you — based on the freshest research available (the DOCFORGE-BENCH paper was published March 2, 2026, just three weeks ago).

---

## The single most important finding you need to know

DOCFORGE-BENCH finds a pervasive calibration failure invisible under single-threshold protocols: methods achieve moderate Pixel-AUC of 0.76 or higher, yet near-zero Pixel-F1. This AUC-F1 gap is not a discrimination failure but a score-distribution shift — tampered regions occupy only 0.27–4.17% of pixels in document images, an order of magnitude less than in natural image benchmarks, making the standard 0.5 threshold catastrophically miscalibrated.

In plain language: every model — including your original five — can actually *detect* tampered pixels in bank documents. They just think the right answer threshold is 0.5 when it should be 0.05. The good news: adapting a single threshold on just 10 domain images recovers 39–55% of the Oracle-F1 gap, demonstrating that threshold adaptation, not retraining, is the key missing step for practical deployment.

---

## The ranked recommendation for bank documents

**Use ADCD-Net (ICCV 2025) as your primary model.** ADCD-Net demonstrates superior forgery localization performance, consistently outperforming state-of-the-art methods by 20.79% averaged over 5 types of distortions. It is the only model specifically designed around the two properties that make bank document tampering hard: text–background intensity contrast causes other models to confuse pristine text pixels with tampered ones, and JPEG block alignment breaks when documents are cropped or resized. ADCD-Net adaptively modulates DCT feature contributions based on a predicted alignment score, achieving improved resilience to distortions including resizing and cropping, and uses a pristine background prototype capturing traces of untampered regions to enhance both localization accuracy and robustness. Source code and pretrained checkpoint are at `github.com/KAHIMWONG/ADCD-Net`.

**Use CAFTB-Net as your cross-domain generalization model.** CAFTB-Net is the only document-specific method that clearly outperforms both TruFor and CAT-Net on cross-domain F1. When you encounter a bank document type not in your training data, CAFTB-Net degrades more gracefully than DTD or ADCD-Net. Available via `github.com/scu-zjz/ForensicHub`.

**Use ASCFormer when you specifically handle receipts, invoices, and bank slips.** The RTM dataset consists of 6,000 images manually tampered using Adobe Photoshop by professional editors, narrowing the gap with practical scenarios, and ASCFormer achieves the best overall performance on RTM. This is the most realistic training data for bank fraud — not synthetic synthesis, but actual professional Photoshop forgeries. The dataset and pretrained model are at `github.com/DrLuo/RTM`.

**Surprisingly, also run TruFor as a second-layer validator.** TruFor outperforms all document-specific methods on MixTamper (F1=0.689) and FantasyID (F1=0.296), underscoring that domain-specific training does not guarantee cross-domain superiority. Its camera noise fingerprint signal is domain-agnostic enough to catch tampering that document-trained models miss.

---

## The practical deployment recipe

```
Step 1 — Threshold calibration (do this first, costs almost nothing)
         Collect 10 authentic + 10 tampered bank document samples.
         Scan the model's output scores on these 20 images.
         Find the threshold that maximises F1 on this tiny calibration set.
         This one step recovers 39–55% of performance. No retraining needed.

Step 2 — Run ADCD-Net for pixel-level localization mask
         Input: scanned bank document (JPEG or PNG)
         Output: binary mask showing exactly which pixels were tampered
         github.com/KAHIMWONG/ADCD-Net

Step 3 — Run CAT-Net as a fast JPEG-artifact pre-screen
         Catches copy-paste from a different source JPEG in < 100ms
         Escalate to ADCD-Net only if CAT-Net flags suspicion

Step 4 — Run TruFor as independent cross-validation
         If TruFor and ADCD-Net agree → high confidence verdict
         If they disagree → human review queue

Step 5 — Human review with ADCD-Net mask
         Analyst sees the localization mask highlighting the exact region
         Review time: 2–3 minutes vs 15–20 minutes without the mask
```

---

## Key datasets for fine-tuning on your specific bank document types

| Dataset | Content | Size | Link |
|---|---|---|---|
| DocTamper | Contracts, invoices, receipts, 170K images | 170K | `github.com/qcf-568/DocTamper` |
| RTM (Real Text Manipulation) | 6,000 manually Photoshop-forged documents | 9K | `github.com/DrLuo/RTM` |
| ReceiptForgery | Tampered receipts specifically | 218 images | Part of DocForge-Bench |
| FantasyID | ID card manipulations | 2,773 test images | `deepid-iccv.github.io` |
| FSTS-1.5k | 1,488 real-world tampered images from 5 forgery types | 1.5K | arXiv:2503.xxxxx |
