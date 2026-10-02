# Durham E-Theses Prototype Search Tool
A search tool to help researchers find information buried in a collection of 14000+ theses from over 100 years of research at Durham University.

## How it works
* Enter a search term, optional date range and author name in the web UI (or send a request to the API directly).  
* Uses FAISS similarity search with sentence transformers ([`sentence-transformers/all-mpnet-base-v2`](https://huggingface.co/sentence-transformers/all-mpnet-base-v2)) to find the theses in the collection most relevant to the search, using title and abstract data.
* Author filtering is performed using fuzzy search.
* Results are returned in order of similarity score.
* Summarises the returned theses using AI (Gemini). Also provides the ability to ask Gemini questions using the theses as a source to extract specific information (with page numbers).
* API is built using Python/FastAPI, frontend is served by a separate NodeJS server, providing the option for others to build their own frontend.

Note: due to limitations and rate limits imposed by Gemini's free tier, AI summaries are only provided for individual theses when the button is clicked. In production, these will be generated automatically for the top x results.   
Note 2: Due to context limits, the system currently only allows use of a single thesis as a source when asking questions. In the future, either using a paid model with a higher context limit or rewriting the system to only pass relevant parts of the theses to Gemini's context, this can be upgraded to allow the use of multiple theses to provide more in-depth information extraction for researchers.


## First-time Setup (only has to be done once):

### Requirements
* **Durham E-Theses Database**: Place in `python/db` folder
* **durham_thesis.index**: Place in `python` folder (no sub-folders)
* **durham_thesis_ids.npy**: Place in `python` folder (no sub-folders)

(These files are not included in the repository due to size. These can be built from the Admin page, although it will take some time. Since the database is built using web scraping, it is recommended to *ask for permission* from Durham University before building the database, and/or impose a rate limit on the scraper to avoid overloading their servers. Scraper is working as of 04/2026, may break in the future - feel free to make a PR with any updates.)

* **[NodeJS](https://nodejs.org/en/download)**
* **[Python 3.11](https://www.python.org/downloads/latest/python3.11/)**

### Setup
#### Node.js
- `cd nodejs`
- `npm install`

#### Python
- `cd python`
- `python -m venv .venv`
- MacOS/Linux: `source .venv/bin/activate`, Windows: `./.venv/Scripts/activate`
- `pip install -r requirements.txt`
Note: If your device does not have a CUDA-capable GPU, use `requirements_minimum.txt` instead. CUDA is *highly recommended* for building the model index files.

#### Environment Variables
Create a file in the `python` folder called `.env` and insert the following lines:
```
TESSDATA_PREFIX = <...>/tessdata  
DB_PATH = <...>/db.db  
GEMINI_API_KEY = <your_gemini_api_key>  
SECRET_KEY = <KEYHERE>
```
Replacing `<...>` with the correct paths to the files on your machine, and `<your_gemini_api_key>` with your Gemini API key  
Tessdata can be cloned from https://github.com/tesseract-ocr/tessdata and is required to add full thesis texts to the database.  
A Gemini API key can be generated from https://aistudio.google.com/api-keys
Default db path is ./db/db.db, to guarantee functionality on the installed system set this to the full path of the db file in your filesystem.

Replace `<KEYHERE>` with your secret key - recommended: randomly generated 32 character string. This should NOT be made public, as this key enables admin access to the system.


## Running the code:

### Windows
From the root project directory, in Windows PowerShell, run:
- `./start`
Alternatively, in Command Prompt, run:
- `call ./start`

### Mac/Linux
From root project directory, in a Bash terminal, run:
- `chmod u+x ./start.sh`
- `./start.sh`

### Creating an Admin User
- `cd python`
- `python create_admin.py <username> <password>`
- Login with the credentials at http://localhost:8080/login

### Running tests
API endpoint tests are in `test_main.py` These purely test the functionality of the API, while mocking external function calls and DB queries.   
Running tests:
- Ensure Python modules `pytest` and `httpx` are installed in the virtual environment (they are in `requirements.txt` now)

Windows Powershell:
- `./test`

Bash:
- `./test.sh`


## Maintenance
On the Admin Page:
- `Update DB`: Calls the scraper to add any new theses to the system's database. All operations are performed on a copy of the DB to prevent disruption to the live system.
- `Load Updated DB`: Swaps the copy created by `Update DB` to the live system and deletes the copy.
- `Rebuild Index`: Rebuilds the database index used by the model. This must be completed after every database update for the new entries to be searchable. (Note: perform AFTER loading the updated DB into the system). Saves the updated index files into a copy to prevent disruption to the live system.
- `Load New Index`: Swaps the copy created by `Rebuild Index` to the live system and deletes the copy. Note: this will cause some API downtime (<1min) while the model is reinitialised.
Notes:
- Ensure the DB is kept up to date - can either schedule updates or integrate properly with Durham E-theses
- Ensure the Gemini API Key always has sufficient credits, otherwise AI functionality will break.


## Possible Future Developments
- Integrate properly into a thesis collection (i.e. without web scraping)
- Scale up to a larger thesis repository, such as [EThOS](https://ethos.bl.uk/)
- Allow direct DB record modification from the admin dashboard
- Rewrite the Gemini summarisation components so that only relevant pages of the theses are passed to Gemini, allowing multiple theses' content to be included within the context limit, and use a better (paid) model with a larger context limit.

