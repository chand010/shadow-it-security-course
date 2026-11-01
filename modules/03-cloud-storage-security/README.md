# Module 03 — Cloud Storage Security

This module assesses cloud collaboration and storage controls using an isolated Nextcloud lab and fictional court records.

## Lesson

- [Lesson 03 — Cloud Storage Security and Sharing Governance](LESSON-03.md)

## Lab assets

- [Nextcloud Compose file](../../labs/nextcloud/compose.yaml)
- [Sharing review template](../../labs/nextcloud/SHARING_REVIEW_TEMPLATE.csv)

## Safety

The Compose example binds the web interface to loopback, does not publish the database port, uses a private Docker network and requires local secrets in an untracked .env file. It is a training environment only, not a production architecture. Review existing containers and volumes before deployment. Use synthetic information only.
