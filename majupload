import sys
import os
import pickle
from PyQt6.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QFileDialog, QLabel, QLineEdit, QProgressBar, QTextBrowser
from PyQt6.QtCore import QThread, pyqtSignal, Qt
from PyQt6.QtGui import QFont
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
import google_auth_oauthlib.flow
import os
import json
from playwright.sync_api import sync_playwright

SCOPES = ["https://www.googleapis.com/auth/youtube.upload"]
CREDENTIALS_FILE = "credentials.json"
TOKEN_FILE = "token.pickle"

class UploadThread(QThread):
    progress = pyqtSignal(int)
    finished = pyqtSignal(str)

    def __init__(self, file_path, title, description, youtube):
        super().__init__()
        self.file_path = file_path
        self.title = title
        self.description = description
        self.youtube = youtube

    def run(self):
        request = self.youtube.videos().insert(
            part="snippet,status", 
            body={
                "snippet": {
                    "title": self.title,
                    "description": self.description,
                    "categoryId": "22"
                },
                "status": {
                    "privacyStatus": "unlisted",  # Доступ по ссылке
                    "madeForKids": False,
                    "selfDeclaredMadeForKids": False
                }
            },
            media_body=MediaFileUpload(self.file_path, chunksize=-1, resumable=True)
        )
        self.progress.emit(50)
        response = request.execute()
        self.progress.emit(100)
        self.finished.emit(f"https://www.youtube.com/watch?v={response['id']}")

class ForumComplaintBot:
    def __init__(self):
        self.username = ""  # Замените на ваш логин
        self.password = ""  # Замените на ваш пароль
        self.login_url = "https://forum.majestic-rp.ru/login"
        self.complaint_url = "https://forum.majestic-rp.ru/forums/zhaloby-na-igrokov.1148/post-thread"
        self.cookies_file = 'cookies.json'  # Файл для сохранения/загрузки куков

    def login(self, page):
        """Метод для авторизации"""
        if os.path.exists(self.cookies_file):
            print("Загружаем куки...")
            with open(self.cookies_file, 'r') as cookies_file:
                cookies = json.load(cookies_file)
                for cookie in cookies:
                    page.context.add_cookies([cookie])
            print("Куки загружены!")
            page.goto(self.login_url)
            page.wait_for_load_state('domcontentloaded')
            return

        print("Начинаем авторизацию...")
        page.goto(self.login_url)
        page.wait_for_selector("input[name='login']")
        page.fill("input[name='login']", self.username)
        page.fill("input[name='password']", self.password)
        page.click("button[type='submit']")
        page.wait_for_load_state('domcontentloaded')

        current_url = page.url
        print(f"Текущий URL после авторизации: {current_url}")

        content = page.content()
        print("Контент страницы после авторизации:")
        print(content)

        cookies = page.context.cookies()
        with open(self.cookies_file, 'w') as cookies_file:
            json.dump(cookies, cookies_file)
        print("Куки сохранены!")

    def submit_complaint(self, page, video_link):
        """Метод для подачи жалобы"""
        print("Переходим на страницу подачи жалобы...")
        page.goto(self.complaint_url)

        print("Ждем появления полей формы...")
        page.wait_for_selector("textarea[name='custom_fields[6]']")  
        print("Поля формы появились!")

        print("Заполняем поля формы...")
        page.evaluate('''(args) => {
    const element = document.querySelector(args.selector);
    if (element) {
        element.innerHTML = args.text;
    }
}''', {'selector': '.fr-element[contenteditable="true"]', 'text': '.'})
        page.fill("textarea[name='title']", "Жалоба на #")
        page.fill("input[name='custom_fields[1]']", "")
        page.fill("input[name='custom_fields[2]']", "")
        page.fill("input[name='custom_fields[4]']", "Дата и время нарушения")
        page.fill("textarea[name='custom_fields[5]']", "Краткое описание ситуации")
        page.fill("textarea[name='custom_fields[6]']", video_link)  #
        print("Поля формы заполнились!")

    def run(self, video_link=None):
        """Запуск всех операций: авторизация и подготовка формы"""
        with sync_playwright() as p:
            browser = p.chromium.launch(headless=False)
            page = browser.new_page()

            self.login(page)
            if video_link:
                self.submit_complaint(page, video_link)  

            input("Закройте браузер, когда закончите, и нажмите Enter для завершения программы.")

class YouTubeUploader(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle("YouTube Uploader")
        self.resize(600, 400)
        self.setStyleSheet("""
            background-color: #21252B;
            color: #ABB2BF;
            font-family: 'Segoe UI', sans-serif;
        """)

        layout = QVBoxLayout()

        self.label = QLabel("Выберите видео для загрузки")
        self.label.setFont(QFont("Segoe UI", 14))
        self.label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        layout.addWidget(self.label)

        self.title_input = QLineEdit(self)
        self.title_input.setPlaceholderText("Название видео")
        self.title_input.setStyleSheet("""
            background-color: #2C313C;
            color: white;
            padding: 10px;
            border: 1px solid #61AFEF;
            border-radius: 8px;
        """)
        layout.addWidget(self.title_input)

        self.desc_input = QLineEdit(self)
        self.desc_input.setPlaceholderText("Описание видео")
        self.desc_input.setStyleSheet("""
            background-color: #2C313C;
            color: white;
            padding: 10px;
            border: 1px solid #61AFEF;
            border-radius: 8px;
        """)
        layout.addWidget(self.desc_input)

        self.upload_button = QPushButton("Выбрать видео", self)
        self.upload_button.setStyleSheet("""
            background-color: #61AFEF;
            color: white;
            padding: 12px;
            border-radius: 10px;
            border: none;
            font-size: 16px;
            transition: background-color 0.3s;
        """)
        self.upload_button.setCursor(Qt.CursorShape.PointingHandCursor)
        self.upload_button.clicked.connect(self.select_file)
        layout.addWidget(self.upload_button)

        self.upload_video_button = QPushButton("Загрузить видео", self)
        self.upload_video_button.setStyleSheet("""
            background-color: #98C379;
            color: white;
            padding: 12px;
            border-radius: 10px;
            border: none;
            font-size: 16px;
            transition: background-color 0.3s;
        """)
        self.upload_video_button.setEnabled(False)
        self.upload_video_button.setCursor(Qt.CursorShape.PointingHandCursor)
        self.upload_video_button.clicked.connect(self.upload_video)
        layout.addWidget(self.upload_video_button)

        self.skip_button = QPushButton("Пропустить загрузку", self)  
        self.skip_button.setStyleSheet("""
            background-color: #D19A66;
            color: white;
            padding: 12px;
            border-radius: 10px;
            border: none;
            font-size: 16px;
            transition: background-color 0.3s;
        """)
        self.skip_button.setCursor(Qt.CursorShape.PointingHandCursor)
        self.skip_button.clicked.connect(self.skip_upload)
        layout.addWidget(self.skip_button)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)
        self.progress_bar.setStyleSheet("""
            QProgressBar {
                background-color: #3E4451;
                border-radius: 8px;
            }
            QProgressBar::chunk {
                background-color: #98C379;
                border-radius: 8px;
            }
        """)
        layout.addWidget(self.progress_bar)

        self.link_output = QTextBrowser(self)
        self.link_output.setStyleSheet("""
            background-color: #2C313C;
            color: #61AFEF;
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #61AFEF;
        """)
        self.link_output.setOpenExternalLinks(True)
        layout.addWidget(self.link_output)

        self.copy_button = QPushButton("Скопировать ссылку", self)
        self.copy_button.setStyleSheet("""
            background-color: #98C379;
            color: white;
            padding: 10px;
            border-radius: 8px;
            border: none;
            font-size: 16px;
        """)
        self.copy_button.clicked.connect(self.copy_link)
        self.copy_button.setVisible(False)
        self.copy_button.setCursor(Qt.CursorShape.PointingHandCursor)
        layout.addWidget(self.copy_button)

        self.setLayout(layout)

    def get_authenticated_service(self):
        creds = None
        if os.path.exists(TOKEN_FILE):
            with open(TOKEN_FILE, "rb") as token:
                creds = pickle.load(token)
        if not creds or not creds.valid:
            flow = google_auth_oauthlib.flow.InstalledAppFlow.from_client_secrets_file(CREDENTIALS_FILE, SCOPES)
            creds = flow.run_local_server(port=8080)
            with open(TOKEN_FILE, "wb") as token:
                pickle.dump(creds, token)
        return build("youtube", "v3", credentials=creds)

    def select_file(self):
        file_path, _ = QFileDialog.getOpenFileName(self, "Выберите видеофайл", "", "Видео (*.mp4 *.mov *.avi)")
        if file_path:
            self.file_path = file_path
            self.upload_video_button.setEnabled(True)  

    def upload_video(self):
        youtube = self.get_authenticated_service()
        title = self.title_input.text() or "Без названия"
        description = self.desc_input.text() or "Описание отсутствует"

        self.upload_thread = UploadThread(self.file_path, title, description, youtube)
        self.upload_thread.progress.connect(self.progress_bar.setValue)
        self.upload_thread.finished.connect(self.display_link)
        self.upload_thread.start()

    def display_link(self, link):
        self.link_output.setText(f'<a href="{link}">{link}</a>')
        self.copy_button.setVisible(True)
        self.link_to_copy = link

    def skip_upload(self):
        video_link = "https://www.youtube.com/watch?v=dQw4w9WgXcQ" 
        self.submit_complaint(video_link)

    def submit_complaint(self, video_link):
        bot = ForumComplaintBot()
        bot.run(video_link=video_link)  

    def copy_link(self):
        clipboard = QApplication.clipboard()
        clipboard.setText(self.link_to_copy)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = YouTubeUploader()
    window.show()
    sys.exit(app.exec())
