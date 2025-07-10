### get_papers/__init__.py
# Empty init file


### get_papers/fetch.py
import requests
import xml.etree.ElementTree as ET
from typing import List

def fetch_pubmed_ids(query: str) -> List[str]:
    r = requests.get("https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi", params={
        "db": "pubmed", "term": query, "retmax": 100, "retmode": "json"
    })
    return r.json()["esearchresult"].get("idlist", [])

def fetch_details(pubmed_ids: List[str]) -> List[ET.Element]:
    if not pubmed_ids: return []
    r = requests.get("https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi", params={
        "db": "pubmed", "id": ",".join(pubmed_ids), "retmode": "xml"
    })
    return ET.fromstring(r.content).findall(".//PubmedArticle")


### get_papers/filter.py
import re
from typing import List, Tuple

NON_ACADEMIC = ["pharma", "biotech", "inc", "ltd", "corp", "gmbh", "llc", "solutions"]
ACADEMIC = ["university", "college", "hospital", "institute"]

def is_non_academic(aff: str) -> bool:
    aff = aff.lower()
    return not any(k in aff for k in ACADEMIC) and any(k in aff for k in NON_ACADEMIC)

def extract_info(article) -> Tuple[List[str], List[str], str]:
    authors = article.findall(".//Author")
    nauthors, affs, email = [], [], ""
    for a in authors:
        name = f"{a.findtext('ForeName', '')} {a.findtext('LastName', '')}".strip()
        aff = a.findtext("AffiliationInfo/Affiliation")
        if aff and is_non_academic(aff):
            nauthors.append(name)
            affs.append(aff)
            if not email and (m := re.search(r"[\w.-]+@[\w.-]+", aff)):
                email = m.group()
    return nauthors, affs, email


### get_papers/utils.py
from typing import List, Dict
import csv

def write_csv(data: List[Dict[str, str]], filename: str):
    with open(filename, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=data[0].keys())
        writer.writeheader(); writer.writerows(data)

def print_table(data: List[Dict[str, str]]):
    for row in data:
        print("\n".join([f"{k}: {v}" for k, v in row.items()]), "\n" + "-"*40)


### get_papers/cli.py
import typer
from get_papers.fetch import fetch_pubmed_ids, fetch_details
from get_papers.filter import extract_info
from get_papers.utils import write_csv, print_table

app = typer.Typer()

@app.command()
def main(query: str, file: str = None):
    ids = fetch_pubmed_ids(query)
    articles = fetch_details(ids)
    results = []
    for art in articles:
        pmid = art.findtext(".//PMID")
        title = art.findtext(".//ArticleTitle")
        date = art.findtext(".//PubDate/Year") or "Unknown"
        authors, affs, email = extract_info(art)
        if authors:
            results.append({
                "PubmedID": pmid,
                "Title": title,
                "Publication Date": date,
                "Non-academic Author(s)": "; ".join(authors),
                "Company Affiliation(s)": "; ".join(affs),
                "Corresponding Author Email": email
            })
    if not results:
        print("No non-academic authors found.")
        return
    if file:
        write_csv(results, file)
        print(f"Saved to {file}")
    else:
        print_table(results)

if __name__ == "__main__":
    app()
