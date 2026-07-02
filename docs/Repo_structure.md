# 📁 Project Directory Structure

```text
Athena/
│
├── README.md
├── LICENSE
├── .gitignore
├── docker-compose.yml
├── .env.example
│
├── docs/
│   ├── architecture.md
│   ├── roadmap.md
│   ├── api.md
│   ├── database.md
│   ├── prompts.md
│   └── meeting-notes.md
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── styles/
│   │   └── utils/
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── agents/
│   │   ├── core/
│   │   ├── models/
│   │   ├── services/
│   │   ├── database/
│   │   ├── schemas/
│   │   ├── prompts/
│   │   ├── utils/
│   │   └── main.py
│   │
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── data/
│   ├── syllabus/
│   │   ├── chemistry/
│   │   ├── physics/
│   │   └── biology/
│   │
│   ├── past_papers/
│   │   ├── chemistry/
│   │   ├── physics/
│   │   └── biology/
│   │
│   ├── examiner_reports/
│   └── mark_schemes/
│
├── embeddings/
│   ├── chroma/
│   └── faiss/
│
├── scripts/
│   ├── ingest_data.py
│   ├── build_vectors.py
│   ├── parse_questions.py
│   ├── generate_metadata.py
│   └── clean_database.py
│
├── pdf_generator/
│   ├── templates/
│   └── generator.py
│
├── notebooks/
│   ├── experimentation.ipynb
│   └── evaluation.ipynb
│
├── evaluation/
│   ├── benchmark.py
│   ├── test_questions.json
│   └── results/
│
└── .github/
    ├── workflows/
    └── ISSUE_TEMPLATE/
```
