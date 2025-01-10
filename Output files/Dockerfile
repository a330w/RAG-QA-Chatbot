FROM python:3.9
WORKDIR /python-image
ADD requirements.txt .
RUN pip install -r requirements.txt
ADD rag_chatbot.py .
ENV PYTHONUNBUFFERED=1
CMD ["python", "rag_chatbot.py"]