# happyfox
import os
import pickle
import google.auth
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

# If modifying the scope, delete the token.pickle file.
SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']

def authenticate():
    creds = None
    token_path = 'token.pickle'

    if os.path.exists(token_path):
        with open(token_path, 'rb') as token:
            creds = pickle.load(token)

    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)

        # Save the credentials for the next run
        with open(token_path, 'wb') as token:
            pickle.dump(creds, token)

    return creds

def fetch_emails():
    creds = authenticate()
    service = build('gmail', 'v1', credentials=creds)

    # Call the Gmail API to fetch the list of emails
    results = service.users().messages().list(userId='me').execute()
    messages = results.get('messages', [])

    if not messages:
        print('No emails found.')
    else:
        print('Emails:')
        for message in messages:
            email = service.users().messages().get(userId='me', id=message['id']).execute()
            print(email['snippet'])

if __name__ == '__main__':
    fetch_emails()

