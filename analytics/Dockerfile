FROM python:3.11-slim

WORKDIR /src

# Install dependencies first so this layer is cached between code changes
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV APP_PORT=5153
EXPOSE 5153

CMD ["python", "app.py"]
