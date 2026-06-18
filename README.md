# Unified Platform Course Docs

Документация проекта Unified Platform для зачета лабораторных работ по дисциплине "Средства Web-программирования".

## Локальная проверка

```bash
uvx --with mkdocs-material mkdocs serve -a 127.0.0.1:8001
```

## Публикация на GitHub Pages

1. Создать новый репозиторий на GitHub, например `unified-platform-course-docs`.
2. Залить содержимое этой папки в репозиторий.
3. В GitHub открыть `Settings` -> `Pages`.
4. В `Build and deployment` выбрать `Source: GitHub Actions`.
5. Дождаться выполнения workflow `Deploy MkDocs to GitHub Pages`.
6. Ссылка будет вида:

```text
https://<github-username>.github.io/unified-platform-course-docs/
```
