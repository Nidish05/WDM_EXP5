### EX5 Information Retrieval Using Boolean Model in Python
### DATE: 23/11/2026
### Name: Nidishkumar S
### Reg No: 212224040218
### AIM: To implement Information Retrieval Using Boolean Model in Python.
### Description:
<div align = "justify">
The Boolean model in Information Retrieval (IR) is a fundamental model used for searching and retrieving information from a collection of documents. It operates on the principles of set theory and logic, where documents are represented as sets of terms or words, and queries are expressed as Boolean expressions using logical operators such as AND, OR, and NOT.
  
### Procedure:
1. ***Initialize the BooleanRetrieval class:*** The BooleanRetrieval class is defined to manage the indexing and searching of documents.
2. ***Constructor and Index Initialization:*** The class constructor initializes an empty index to store the inverted index mapping terms to documents.
3. ***Indexing Documents:***
    <p> a) The index_document method is responsible for indexing documents.
    <p> b) Tokenize the text content of documents, converting them into lowercase terms.
    <p> c) For each term in the document, it adds an entry in the index, associating the term with the document ID. </p>
4. ***Fetch Web Page Text:***
    <p>a) The fetch_webpage_text method uses the requests library to fetch content from a given URL.
    <p>b) Extract text content from the fetched HTML using BeautifulSoup.
    <p>c) The extracted text is returned for further processing.
5. ***Boolean Search:***
    <p>a) The boolean_search method performs Boolean searches on the indexed documents.
    <p>b) Tokenize the input query and iterates through its terms.
    <p>c) For each term in the query, it retrieves documents containing that term and performs Boolean operations (AND, OR, NOT) based on the query's structure.

### Program:
```python

import re
import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup


class BooleanRetrieval:
    def __init__(self):
        self.index = {}
        self.documents_matrix = None
        self.doc_ids = []

    def index_document(self, doc_id, text):
        terms = text.lower().split()
        print("Document -", doc_id, terms)

        if doc_id not in self.doc_ids:
            self.doc_ids.append(doc_id)

        for term in terms:
            if term not in self.index:
                self.index[term] = set()
            self.index[term].add(doc_id)

    def create_documents_matrix(self, documents):
        terms = list(self.index.keys())
        num_docs = len(documents)
        num_terms = len(terms)

        self.doc_ids = list(documents.keys())
        self.documents_matrix = np.zeros((num_docs, num_terms), dtype=int)

        for i, (doc_id, text) in enumerate(documents.items()):
            doc_terms = text.lower().split()
            for term in doc_terms:
                if term in self.index:
                    term_id = terms.index(term)
                    self.documents_matrix[i, term_id] = 1

    def print_documents_matrix_table(self):
        df = pd.DataFrame(
            self.documents_matrix,
            index=self.doc_ids if self.doc_ids else None,
            columns=self.index.keys()
        )
        print(df)

    def print_all_terms(self):
        print("All terms in the documents:")
        print(list(self.index.keys()))

    def fetch_webpage_text(self, url):
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        soup = BeautifulSoup(response.text, "html.parser")
        for script_or_style in soup(["script", "style"]):
            script_or_style.decompose()
        return " ".join(soup.get_text().split())

    def _all_doc_ids(self):
        if self.doc_ids:
            return set(self.doc_ids)
        all_docs = set()
        for doc_set in self.index.values():
            all_docs.update(doc_set)
        return all_docs

    def _tokenize_query(self, query):
        raw_tokens = re.findall(
            r'\(|\)|\bAND\b|\bOR\b|\bNOT\b|[^\s()]+',
            query.strip(),
            flags=re.IGNORECASE
        )
        tokens = []
        for tok in raw_tokens:
            upper = tok.upper()
            if upper in ("AND", "OR", "NOT", "(", ")"):
                tokens.append(upper)
            else:
                tokens.append(tok.lower())
        return tokens

    def _to_postfix(self, tokens):
        precedence = {"NOT": 3, "AND": 2, "OR": 1}
        right_associative = {"NOT"}
        output = []
        stack = []

        for token in tokens:
            if token == "(":
                stack.append(token)
            elif token == ")":
                while stack and stack[-1] != "(":
                    output.append(stack.pop())
                if stack and stack[-1] == "(":
                    stack.pop()
            elif token in precedence:
                while (
                    stack
                    and stack[-1] != "("
                    and stack[-1] in precedence
                    and (
                        precedence[stack[-1]] > precedence[token]
                        or (
                            precedence[stack[-1]] == precedence[token]
                            and token not in right_associative
                        )
                    )
                ):
                    output.append(stack.pop())
                stack.append(token)
            else:
                output.append(token)

        while stack:
            if stack[-1] != "(":
                output.append(stack.pop())
            else:
                stack.pop()

        return output

    def boolean_search(self, query):
        tokens = self._tokenize_query(query)
        if not tokens:
            return []

        postfix = self._to_postfix(tokens)
        universe = self._all_doc_ids()
        stack = []

        for token in postfix:
            if token == "NOT":
                operand = stack.pop() if stack else set()
                stack.append(universe - operand)
            elif token == "AND":
                right = stack.pop() if stack else set()
                left = stack.pop() if stack else set()
                stack.append(left & right)
            elif token == "OR":
                right = stack.pop() if stack else set()
                left = stack.pop() if stack else set()
                stack.append(left | right)
            else:
                stack.append(set(self.index.get(token, set())))

        return sorted(stack[-1]) if stack else []


if __name__ == "__main__":
    indexer = BooleanRetrieval()

    documents = {
        1: "Python is a programming language",
        2: "Information retrieval deals with finding information",
        3: "Boolean models are used in information retrieval"
    }

    for doc_id, text in documents.items():
        indexer.index_document(doc_id, text)

    indexer.create_documents_matrix(documents)
    indexer.print_documents_matrix_table()
    indexer.print_all_terms()

    query = input("Enter your boolean query: ")
    results = indexer.boolean_search(query)
    if results:
        print(f"Results for '{query}': {results}")
    else:
        print("No results found for the query.")
```

### Output:
<img width="887" height="320" alt="image" src="https://github.com/user-attachments/assets/e67b37d4-b297-4704-9e40-503dfb75bbd6" />
<img width="456" height="53" alt="image" src="https://github.com/user-attachments/assets/674c78da-55c7-4936-b518-9f6701f8f6be" />
<img width="588" height="46" alt="image" src="https://github.com/user-attachments/assets/8c1766b1-4692-4e34-8fb0-455595b4a9db" />

### Result:
Thus, the Information Retrieval system using the Boolean Model was successfully implemented in Python.
