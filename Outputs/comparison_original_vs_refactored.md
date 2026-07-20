# Comparison: Original vs. Refactored

## Step 6 / Compare Behavior

---

## Valid Data: Full Batch

**Command:**
```python
# Original (product_listing_generator_original.ipynb, Step 6)
for idx, row in products_df.iterrows():
    price = float(row["Selling Price"].replace("$", "").replace(",", ""))
    listing = generate_product_listing(...)
```
```python
# Refactored (product_listing_generator_refactored.ipynb, Step 6)
for idx, row in products_df.iterrows():
    price = parse_price(row["Selling Price"])
    listing = generate_product_listing(...)
```

**Output:**
```
Original:   BATCH COMPLETE: 5/5 succeeded, 0 failed
Refactored: BATCH COMPLETE: 5/5 succeeded, 0 failed
```

**Observation:**
Same live dataset (`cvnberk/amazon-products`, `train[:5]`), same success rate, same listing fields (`title`, `description`, `features`, `keywords`). The refactored version adds one new field, `is_high_res`, which the original never produced — see below.

---

## Sample Listing Comparison (Product id 0)

**Command:** `generate_product_listing(...)` on the same product, same image, same price/category, in both notebooks.

**Output (original):**
```json
{
  "title": "Craft-tastic Empower Poster Craft Kit - Inspire Creativity",
  ...
}
```

**Output (refactored):**
```json
{
  "title": "Craft-tastic Empower Poster Craft Kit – Inspire & Create!",
  ...
  "is_high_res": true
}
```

**Observation:**
Both listings describe the same product accurately (colors, dimensions, affirmation words) — wording differs slightly because the model isn't deterministic call to call, not because the refactor changed behavior. The refactored version's `is_high_res: true` field is new: it confirms this image (500x500px) was above the resolution floor the model needs to actually see it, addressing the silent-failure risk documented in `dataset_issue_summary.md` (Week 2/Day 1).

---

## Error Handling: Missing File

**Command:**
```python
load_listings_file("does_not_exist.json")
```

**Output (original):** No equivalent function existed — the original never loaded a listings file back from disk, so this failure mode was untested.

**Output (refactored):**
```
Caught expected error: load_listings_file: no file found at 'does_not_exist.json'. Check the path is correct and the file has been generated.
```

**Observation:** New capability in the refactor, not present in the original at all.

---

## Error Handling: Invalid JSON

**Command:** Malformed JSON text passed through `parse_listing_response`.

**Output (original):**
```python
except json.JSONDecodeError as e:
    print(f"JSON parsing failed: {e}")
    listing = {"raw_response": raw_content}
```

**Output (refactored):**
```
parse_listing_response: JSON parsing failed at line 1, col 5: Expecting value
```

**Observation:** Both catch the error and avoid crashing. The refactored message adds the function name and the exact line/column of the failure — the original's message only had the raw exception text.

---

## Error Handling: Invalid Product Data

**Command:** `validate_listing({"title": "A"})` — missing required fields.

**Output (original):** No equivalent validation existed — Pydantic was imported but never used, so a malformed listing would have passed through silently.

**Output (refactored):**
```
Caught expected error: validate_listing: model response did not match the expected listing shape: 3 validation errors for ProductListing
description
  Field required [type=missing, input_value={'title': 'A'}, input_type=dict]
features
  Field required [type=missing, input_value={'title': 'A'}, input_type=dict]
keywords
  Field required [type=missing, input_value={'title': 'A'}, input_type=dict]
. Check the prompt still requests title, description, features, and keywords.
```

**Observation:** New capability — names every missing/invalid field individually, plus a suggested fix.

---

## Error Handling: Network Failure

**Command:** `encode_image_to_base64("https://this-domain-does-not-exist-12345.example/image.jpg")`

**Output (original):** No handling — an unreachable URL would raise a raw, unlabeled `requests.exceptions.ConnectionError` traceback.

**Output (refactored):**
```
Caught expected error: encode_image_to_base64: could not connect to https://this-domain-does-not-exist-12345.example/image.jpg. Check the URL and your network connection.
```

**Observation:** Refactored version names the function, the URL, and what to check. Original would have crashed with a raw traceback and no guidance.

---

## Error Handling: OpenAI API Error

**Command:** `call_vision_api(...)` called with a deliberately invalid API key (`sk-invalid-key-for-testing`), against a separate client so the real client/key was never touched.

**Output (original):** No handling — an authentication failure would raise a raw, unlabeled `openai.AuthenticationError` traceback.

**Output (refactored):**
```
Caught expected error: call_vision_api: OpenAI API error calling gpt-4o-mini: Error code: 401 - {'error': {'message': 'Incorrect API key provided: sk-inval**************ting. You can find your API key at https://platform.openai.com/account/api-keys.', 'type': 'invalid_request_error', 'code': 'invalid_api_key', 'param': None}, 'status': 401}. Check your API key and request parameters.
```

**Observation:** Real 401 from OpenAI's API, correctly caught and wrapped with the function name and a suggestion. The invalid key is masked by OpenAI's own error response and was never a real credential.

---

## Summary: Compare Behavior (Step 6)

**Does the refactored code work the same as the original?**
Yes. Same live dataset, same 5/5 success rate, same core listing fields. The refactor doesn't change what a successful run produces — it adds one new field (`is_high_res`) and changes what happens when something goes wrong.

**Are error messages better?**
Yes, concretely. The original had exactly one handled error path (`JSONDecodeError`, printed with no function name or location) and no handling at all for network failures, file-not-found, or invalid listing shape. The refactored version handles all 5 required error types (`FileNotFoundError`, `JSONDecodeError`, Pydantic `ValidationError`, OpenAI API errors, network errors), and every message names the function, what happened, and what to check — verified in the test cells above.

**Is the code easier to understand?**
Yes. The original's `generate_product_listing` did four jobs in one function: build the prompt, encode the image, call the API, and parse the response — with no way to test any piece independently. The refactored version splits this into four single-responsibility functions (`build_vision_messages`, `call_vision_api`, `parse_listing_response`, `validate_listing`), each with its own test, composed by a slim `generate_product_listing`.

All 5 required error types (`FileNotFoundError`, `JSONDecodeError`, Pydantic `ValidationError`, network errors, OpenAI API errors) are now confirmed with a live test each — see the sections above.
