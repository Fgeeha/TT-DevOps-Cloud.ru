FROM ghcr.io/astral-sh/uv:python3.12-alpine

WORKDIR /app
COPY . /app

ENV AUTHOR=DefaultAuthor
EXPOSE 8000

RUN uv sync --locked

CMD ["uv", "run", "python", "main.py"]
