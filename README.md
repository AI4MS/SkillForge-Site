# SkillForge-Site

Documentation site for the AI4MS group's Materials Computational Science Agent Skills.

This repository is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and serves as the home for our team's Skill construction guidelines, evaluation rubrics, and practical case studies for materials-computation Agent workflows.

Site: <https://AI4MS.github.io/SkillForge-Site/>

## Structure

```
SkillForge-Site/
├── docs/                           # Documentation sources
│   ├── skill-construction-guide.md                    # Skill construction guide (home)
│   └── skill-eval-rubric.md        # Skill evaluation rubric
├── mkdocs.yml                      # MkDocs configuration
└── .github/workflows/              # GitHub Pages deployment
```

## Local preview

```bash
pip install mkdocs-material
mkdocs serve
```

Then open <http://127.0.0.1:8000>.

## Deployment

Pushes to `main` trigger a GitHub Actions workflow that builds and publishes the site to GitHub Pages.

## Contributing

- New docs: add a Markdown file under `docs/` and register it in the `nav` section of `mkdocs.yml`.
- Edits: modify the relevant `.md` file and open a PR.
- Pitfalls and case studies are welcome — please contribute them to the relevant guide.

Maintainer: Gao Yuxiang
