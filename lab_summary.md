# LAB SUMMARY & PROOF | Week 3 | Day 1 | From hacky script to "we could ship this"
## John Adams

## Summary

**Path:** Path 1 — refactored my own code (`product_listing_generator.ipynb`, Week 2/Day 1: API Calling to ChatGPT).

**What was refactored:** The original notebook's `generate_product_listing` function did four jobs at once — built the prompt, encoded the product image, called the OpenAI vision API, and parsed the JSON response — with only one handled error case (`JSONDecodeError`, printed with no function name or location) and no handling at all for network failures, missing files, or malformed listings. Pydantic was imported but never used.

**How it was modularized and error handling improved:** Split into four single-responsibility functions — `build_vision_messages`, `call_vision_api`, `parse_listing_response`, `validate_listing` — each independently tested, composed by a slim `generate_product_listing`. Added explicit handling for all 5 required error types (`FileNotFoundError`, `JSONDecodeError`, Pydantic `ValidationError`, OpenAI API errors, network errors), each confirmed with a live test and each message naming the function, what happened, and a suggested fix. Full before/after detail with real output: `Outputs/comparison_original_vs_refactored.md`.

**Main challenge:** My original Week 2 investigation found that low-resolution product images caused the model to silently ignore the image and write a listing from text metadata alone, with no error to signal it. The refactor's biggest job was turning that into a real, testable signal: `check_image_resolution` flags any image below a resolution floor, and every listing now carries an `is_high_res` field. Tuning that floor was itself a live bug during this refactor — an initial 512px threshold flagged every image in the working dataset (all ~333–500px, genuinely fine) as low-res. Verified the real pixel dimensions directly and reset the floor to 200px, grounded in the original investigation's evidence (60×80px confirmed broken, 313×400px confirmed working).

**What I learned:** This lab's lesson framed refactoring around simplifying code, favoring reusability, and the principle that "code that is not tested is not usable." Splitting `generate_product_listing` into four single-responsibility functions is what made independent unit testing possible in the first place — I couldn't have written a real test for "does image encoding handle a bad URL" while that logic was still fused into one 20-line function doing four jobs. Testing each piece in isolation, before wiring them back together, is also what caught two real bugs early instead of in a pile of end-to-end code: the 512px threshold bug (caught by an assertion, not by staring at output) and the stale-kernel issue (caught because a specific test's output didn't match what the specific function it isolated should have produced). Validation played the same role from the input side — Pydantic's `validate_listing` turns "does the API's response have the right shape" into one reusable, testable check instead of trusting every caller to notice a malformed dict.

**Note on Path 1 vs. the lab's Verification Criteria:** The lab's "Code works with provided JSON data" line and its sample `products.json` / `invalid_products.json` / `malformed.json` files describe Path 2's local-file input. Path 1 (this notebook) reads from a live HuggingFace dataset instead, so there is no local `products.json` in this pipeline — this criterion is not applicable here.

**Explain — first failure mode to monitor if this ran daily:** The silent image-grounding failure, even with `is_high_res` now visible in the output. The flag is informational only — nothing in the pipeline currently blocks or flags a low-res listing before it's saved. Run daily against a rotating dataset, a batch could silently accumulate ungrounded listings again unless something consumes `is_high_res` and acts on it (e.g. routes low-res items to manual review).

---

## Proof of Learning

### Workflow

```
TRIGGER:    Load 5 live products from cvnberk/amazon-products (HuggingFace)
TRANSFORM:  generate_product_listing
              -> build_vision_messages   (prompt + image encoding, flags is_high_res)
              -> call_vision_api         (OpenAI gpt-4o-mini vision call)
              -> parse_listing_response  (JSON parse)
              -> validate_listing        (Pydantic schema check)
ACTION:     Save to Outputs/generated_listings_refactored.json
```

**Code file:** `product_listing_generator_refactored.ipynb`
**Original preserved:** `product_listing_generator_original.ipynb`
**Output file:** `Outputs/generated_listings_refactored.json`

### Execution Trace (Step 6 batch run)

```
Processing product 0: Craft-tastic – Empower Poster – Craft Kit – Design a One-of-a-Kind Inspirational Poster
Processing product 1: Melissa & Doug Dot-to-Dot# & Letter Coloring Pad 3 Pack (ABC Farm, 123 Pets, ABC-123 Wild Animals)
Processing product 2: RPM Rear Shock Tower for The Nitro Slash, Nitro Stampede, Nitro Rustler and Nitro Sport, Black
Processing product 3: Disney Pixar Cars Mini Racers Crank & Crash Derby Playset
Processing product 4: Areaware Cubebot Small
BATCH COMPLETE: 5/5 succeeded, 0 failed
Saved to Outputs/generated_listings_refactored.json
```

### Input Payload (product id 0)

```
product_name: "Craft-tastic – Empower Poster – Craft Kit – Design a One-of-a-Kind Inspirational Poster"
price:        14.47
category:     "Toys & Games | Arts & Crafts | Craft Kits | Paper Craft"
image_url:    "https://images-na.ssl-images-amazon.com/images/I/51FUt3TYecL.jpg"
```

### Output Record (product id 0)

```json
{
  "id": 0,
  "listing": {
    "title": "Craft-tastic Empower Poster Craft Kit – Inspire & Create!",
    "description": "Unleash your creativity with the Craft-tastic Empower Poster Craft Kit! This fun and engaging kit allows you to design a unique, inspirational poster that celebrates positivity and self-empowerment. Featuring a vibrant explosion of colors, including hues of blue, pink, and green, the poster measures 9.75\" x 13.75\" and is adorned with uplifting words like 'Brave,' 'Joyful,' and 'Unique.' Perfect for kids and adults alike, this craft kit encourages self-expression and boosts confidence through art.",
    "features": [
      "Includes all materials needed to create a poster",
      "Vibrant color palette for an eye-catching design",
      "Encourages creativity and self-expression",
      "Perfect for all ages - from kids to adults",
      "Easy-to-follow instructions for a fun crafting experience",
      "Ideal for gifts, school projects, or home decor",
      "Promotes positive affirmations and boosts confidence"
    ],
    "keywords": "Craft-tastic, Empower Poster, craft kit, inspirational poster, arts and crafts, paper craft, creativity, self-expression, DIY art, kids crafts, motivational decor",
    "is_high_res": true
  },
  "error": null
}
```

### Error Handling Evidence

All 5 required error types confirmed with a live test (function name, what happened, suggested fix — full detail in `Outputs/comparison_original_vs_refactored.md`):

```
Missing file:     load_listings_file: no file found at 'does_not_exist.json'. Check the path is correct and the file has been generated.
Invalid JSON:     parse_listing_response: JSON parsing failed at line 1, col 5: Expecting value
Invalid product:  validate_listing: model response did not match the expected listing shape: 3 validation errors for ProductListing ...
Network error:    encode_image_to_base64: could not connect to https://this-domain-does-not-exist-12345.example/image.jpg. Check the URL and your network connection.
OpenAI API error: call_vision_api: OpenAI API error calling gpt-4o-mini: Error code: 401 - {'error': {'message': 'Incorrect API key provided: sk-inval**************ting. ...', 'type': 'invalid_request_error', 'code': 'invalid_api_key'}, 'status': 401}. Check your API key and request parameters.
```

### Verify

Reviewer can inspect `Outputs/generated_listings_refactored.json` directly (5 records, `id`/`listing`/`error` per product) and cross-check any record's `listing.title`/`description` against the source image URL in the notebook's Step 6 dataset load, to confirm the listing was actually grounded in the image.

### Prompt Discipline (Workflow Reviewer Challenge)

Applied against the built pipeline (trigger: dataset load, transform: `generate_product_listing`, destination: `Outputs/generated_listings_refactored.json`):

**Three workflow risks:**
1. The dataset slice `train[:5]` is an index-based cut, not ID-based — if the upstream dataset is ever reordered or updated, "product 0" silently becomes a different product with no warning.
2. `validate_listing` enforces the Pydantic schema, but `generate_product_listing` still returns a raw fallback dict (`{"raw_response": ...}`) when JSON parsing fails outright — two different shapes can reach the output file, and nothing downstream checks for that inconsistency before writing to disk.
3. `is_high_res` is computed and stored, but nothing in the pipeline acts on it — a low-res image still produces and saves a listing exactly like a high-res one; the flag is informational only, not a gate.

**Two trace checks:**
1. Does every record in `Outputs/generated_listings_refactored.json` have either a populated `listing` or a populated `error`, never both null or both filled?
2. Does `is_high_res` appear on every successful listing, confirming the detection ran on every item, not just the first?

**One simplification:** `build_vision_messages` returns a tuple `(messages, is_high_res)` — mixing "what to send" with "a fact about the image" in one return value. A small dataclass or two named return values would read cleaner, though not worth a further refactor this late in the lab.