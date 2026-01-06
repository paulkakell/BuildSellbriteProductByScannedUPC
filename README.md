# Build Product (UPC Scanner to Sellbrite)

A Python utility that scans a UPC barcode using a camera, generates product listing content using OpenAI, estimates pricing from eBay completed listings, then creates a product in Sellbrite.

## What it does

1. Opens a camera feed and waits until it detects a UPC barcode
2. Uses the UPC to generate:
   - Title
   - Description
   - Brand
   - Manufacturer
   - Model number
   - MSRP
   - Category
3. Pulls eBay sold prices (completed items), averages them, then applies a pricing rule
4. Generates a date-prefixed sequential SKU
5. Creates the product in Sellbrite via API

## Repository layout

- `Build_Product.py`
- `sequential_number.txt`
- `.gitignore`
- `README.md`
- `requirements.txt`

Example `sequential_number.txt` contents:

```
0
```

## Requirements

- Python 3.10+ recommended
- A camera accessible by OpenCV
- Sellbrite API credentials
- eBay App ID
- OpenAI API key

System dependencies you may need (varies by OS):

- OpenCV system libs
- ZBar (needed by pyzbar)

If pyzbar fails to import or decode, it is usually missing the ZBar shared library.

## Install

Create and activate a virtual environment, then install dependencies.

Example `requirements.txt`:

```
opencv-python
pyzbar
requests
openai
```

Then:

```
pip install -r requirements.txt
```

## Configure credentials (do not hardcode)

The script currently hardcodes credentials. For GitHub and long-term safety, move secrets into environment variables.

Suggested environment variables:

- `SELLBRITE_API_KEY`
- `SELLBRITE_API_SECRET`
- `EBAY_APP_ID`
- `OPENAI_API_KEY`

Then update `Build_Product.py` to read them via `os.environ`.

## Sellbrite authentication note

The script sets:

- `Authorization: Basic {api_key}:{api_secret}`

That is not standard HTTP Basic auth formatting.

Typical Basic auth formats are either:

1. Use `requests` built-in auth:

```python
requests.post(url, auth=(api_key, api_secret), json=payload)
```

2. Or manually base64 encode `api_key:api_secret`:

```python
import base64
token = base64.b64encode(f"{api_key}:{api_secret}".encode()).decode()
headers["Authorization"] = f"Basic {token}"
```

Check Sellbrite's API docs and align to their expected method.

## Run

1. Ensure `sequential_number.txt` exists in the same directory
2. Verify camera index in `cv2.VideoCapture(1)`
   - Try `0` if you have only one webcam
3. Run:

```
python Build_Product.py
```

Hold the UPC barcode steady in front of the camera until it is detected.

## Operational guidance

- Pricing rule: the script returns half of the average sold price found on eBay.
  Adjust this rule to match your margin, fees, and condition grading.
- UPC inference: generating product metadata purely from a UPC using a language model can be wrong.
  For better accuracy, fetch authoritative UPC catalog data first, then ask the model to rewrite and enrich.

## Troubleshooting

- Camera opens but never detects a UPC:
  - Try a different camera index (0, 1, 2)
  - Improve lighting and focus
  - Accept additional barcode types besides `UPC` if your codes are EAN-13, etc

- ImportError for pyzbar or decode returns nothing:
  - Install ZBar on your OS and confirm it is on the library path

- ValueError when parsing MSRP:
  - The model may return MSRP like `$19.99` or `USD 19.99`
  - Strip currency symbols before converting to float

- Sellbrite returns auth errors:
  - Fix Basic auth formatting as described above
  - Confirm API key permissions

## Security

- Never commit API keys to GitHub
- Add `.env` to `.gitignore` if you use one
- Consider rate limiting and backoff for API calls

## License

Choose a license before publishing (MIT, Apache-2.0, etc). If you do not pick one, many users will treat the code as all rights reserved by default.
