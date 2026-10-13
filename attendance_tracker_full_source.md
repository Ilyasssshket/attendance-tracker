# Полный исходный код проекта

Ниже приведено полное содержимое каждого файла проекта.


## `main.py`

```python
import sys
from PyQt6.QtWidgets import QApplication
from components import AttendanceMainWindow


def main():
    application = QApplication(sys.argv)
    application.setApplicationName("Учёт посещений")
    application.setStyle("Fusion")

    main_window = AttendanceMainWindow()
    main_window.show()

    sys.exit(application.exec())


if __name__ == "__main__":
    main()
```


## `models.py`

```python
from abc import ABC, abstractmethod
from datetime import datetime


class AbstractEntity(ABC):
    """Общий контракт сущностей предметной области."""

    @abstractmethod
    def get_sort_key(self):
        raise NotImplementedError


class User(AbstractEntity):
    def __init__(self, username, password, display_name, role="Пользователь"):
        self._username = username
        self._password = password
        self._display_name = display_name
        self._role = role

    @property
    def username(self):
        return self._username

    @username.setter
    def username(self, value):
        if not isinstance(value, str) or not value.strip():
            raise ValueError("Имя пользователя не может быть пустым.")
        self._username = value.strip()

    @property
    def password(self):
        return self._password

    @password.setter
    def password(self, value):
        if not isinstance(value, str) or len(value) < 1:
            raise ValueError("Пароль должен быть непустой строкой.")
        self._password = value

    @property
    def display_name(self):
        return self._display_name

    @display_name.setter
    def display_name(self, value):
        if not isinstance(value, str) or not value.strip():
            raise ValueError("Отображаемое имя не может быть пустым.")
        self._display_name = value.strip()

    @property
    def role(self):
        return self._role

    @role.setter
    def role(self, value):
        if not isinstance(value, str) or not value.strip():
            raise ValueError("Роль не может быть пустой.")
        self._role = value.strip()

    def get_sort_key(self):
        return self.username.casefold()

    def __eq__(self, other):
        if not isinstance(other, User):
            return NotImplemented
        return self.username.casefold() == other.username.casefold()

    def __lt__(self, other):
        if not isinstance(other, User):
            return NotImplemented
        return self.get_sort_key() < other.get_sort_key()

    def __hash__(self):
        return hash(self.username.casefold())

    def __repr__(self):
        return (
            f"User(username={self.username!r}, "
            f"display_name={self.display_name!r}, role={self.role!r})"
        )

    def __str__(self):
        return f"{self.display_name} ({self.username})"

    def __len__(self):
        return len(self.username)

    def __bool__(self):
        return bool(self.username.strip())

    def __contains__(self, text):
        if not isinstance(text, str):
            return False
        return text.casefold() in self.display_name.casefold()

    def __format__(self, format_spec):
        if format_spec == "username":
            return self.username
        if format_spec == "display":
            return self.display_name
        return str(self)


class LoginSession(AbstractEntity):
    TIME_FORMAT = "%d.%m.%Y %H:%M:%S"

    def __init__(self, user, time_in=None, time_out=None):
        if not isinstance(user, User):
            raise TypeError("Сессия должна быть связана с объектом User.")
        self._user = user
        self._time_in = time_in if time_in is not None else datetime.now()
        self._time_out = time_out

    @property
    def user(self):
        return self._user

    @user.setter
    def user(self, value):
        if not isinstance(value, User):
            raise TypeError("Поле user должно содержать объект User.")
        self._user = value

    @property
    def time_in(self):
        return self._time_in

    @time_in.setter
    def time_in(self, value):
        if not isinstance(value, datetime):
            raise TypeError("Время входа должно иметь тип datetime.")
        self._time_in = value

    @property
    def time_out(self):
        return self._time_out

    @time_out.setter
    def time_out(self, value):
        if value is not None and not isinstance(value, datetime):
            raise TypeError("Время выхода должно иметь тип datetime или None.")
        if value is not None and value < self.time_in:
            raise ValueError("Время выхода не может быть раньше времени входа.")
        self._time_out = value

    def finish(self, timestamp=None):
        if self.time_out is None:
            chosen_timestamp = timestamp if timestamp is not None else datetime.now()
            self.time_out = chosen_timestamp

    def get_sort_key(self):
        return (self.time_in, self.user.username.casefold())

    def format_time_in(self):
        return self.time_in.strftime(self.TIME_FORMAT)

    def format_time_out(self):
        if self.time_out is None:
            return "—"
        return self.time_out.strftime(self.TIME_FORMAT)

    def __eq__(self, other):
        if not isinstance(other, LoginSession):
            return NotImplemented
        return (
            self.user == other.user
            and self.time_in == other.time_in
            and self.time_out == other.time_out
        )

    def __lt__(self, other):
        if not isinstance(other, LoginSession):
            return NotImplemented
        return self.get_sort_key() < other.get_sort_key()

    def __hash__(self):
        return hash((self.user, self.time_in, self.time_out))

    def __repr__(self):
        return (
            f"LoginSession(user={self.user.username!r}, "
            f"time_in={self.time_in!r}, time_out={self.time_out!r})"
        )

    def __str__(self):
        return f"{self.user.username}: {self.format_time_in()} — {self.format_time_out()}"

    def __len__(self):
        return 3

    def __bool__(self):
        return self.time_in is not None

    def __contains__(self, username_fragment):
        if not isinstance(username_fragment, str):
            return False
        return username_fragment.casefold() in self.user.username.casefold()


class CustomMapping:
    """Небольшой маппинг без использования dict в логике приложения."""

    def __init__(self):
        self._keys = []
        self._values = []

    @property
    def keys(self):
        return tuple(self._keys)

    @property
    def values(self):
        return tuple(self._values)

    def set_value(self, key, value):
        if key is None:
            raise ValueError("Ключ маппинга не может быть None.")
        for index, existing_key in enumerate(self._keys):
            if existing_key == key:
                self._values[index] = value
                return
        self._keys.append(key)
        self._values.append(value)

    def get_value(self, key, default=None):
        for index, existing_key in enumerate(self._keys):
            if existing_key == key:
                return self._values[index]
        return default

    def remove_key(self, key):
        for index, existing_key in enumerate(self._keys):
            if existing_key == key:
                self._keys.pop(index)
                return self._values.pop(index)
        return None

    def __contains__(self, key):
        return any(existing_key == key for existing_key in self._keys)

    def __len__(self):
        return len(self._keys)

    def __iter__(self):
        return iter(tuple(self._keys))

    def __eq__(self, other):
        if not isinstance(other, CustomMapping):
            return NotImplemented
        if len(self) != len(other):
            return False
        for key in self._keys:
            if key not in other:
                return False
            if self.get_value(key) != other.get_value(key):
                return False
        return True

    def __repr__(self):
        pairs = []
        for key, value in zip(self._keys, self._values):
            pairs.append(f"{key!r}: {value!r}")
        return "CustomMapping({" + ", ".join(pairs) + "})"

    def __str__(self):
        return f"CustomMapping: {len(self)} записей"

    def __bool__(self):
        return len(self) > 0
```


## `services.py`

```python
from datetime import datetime
from pathlib import Path
from models import User, LoginSession, CustomMapping


class AuthenticationService:
    def __init__(self, users):
        self._users = tuple(users)
        self._users_by_username = CustomMapping()
        for user in self._users:
            self._users_by_username.set_value(user.username.casefold(), user)

    @property
    def users(self):
        return self._users

    def authenticate(self, username, password):
        if not isinstance(username, str) or not isinstance(password, str):
            return None
        normalized_username = username.strip().casefold()
        user = self._users_by_username.get_value(normalized_username)
        if user is None:
            return None
        if user.password != password:
            return None
        return user


class SessionService:
    def __init__(self):
        self._sessions = []
        self._active_sessions = CustomMapping()

    @property
    def sessions(self):
        return tuple(self._sessions)

    def start_session(self, user):
        if not isinstance(user, User):
            raise TypeError("Для создания сессии требуется объект User.")
        active_session = self._active_sessions.get_value(user.username.casefold())
        if active_session is not None:
            return active_session, False

        session = LoginSession(user=user, time_in=datetime.now())
        self._sessions.append(session)
        self._active_sessions.set_value(user.username.casefold(), session)
        return session, True

    def finish_session(self, username):
        if not isinstance(username, str):
            return None
        normalized_username = username.casefold()
        active_session = self._active_sessions.remove_key(normalized_username)
        if active_session is not None:
            active_session.finish(datetime.now())
        return active_session

    def close_all_active_sessions(self):
        active_usernames = list(self._active_sessions.keys)
        for username in active_usernames:
            self.finish_session(username)


class ReportService:
    TIME_FORMAT = "%d.%m.%Y %H:%M:%S"

    def __init__(self, report_path="attendance_report.txt"):
        self._report_path = Path(report_path)

    @property
    def report_path(self):
        return self._report_path

    @report_path.setter
    def report_path(self, value):
        if not isinstance(value, (str, Path)):
            raise TypeError("Путь к отчёту должен быть строкой или Path.")
        self._report_path = Path(value)

    @staticmethod
    def validate_datetime(value):
        if not isinstance(value, datetime):
            raise TypeError("Для форматирования требуется объект datetime.")
        return value.strftime(ReportService.TIME_FORMAT)

    def format_session_line(self, session):
        if not isinstance(session, LoginSession):
            raise TypeError("В отчёт можно добавить только LoginSession.")
        time_in_text = self.validate_datetime(session.time_in)
        if session.time_out is None:
            time_out_text = "—"
        else:
            time_out_text = self.validate_datetime(session.time_out)
        return f"{session.user.username} | {time_in_text} | {time_out_text}"

    def build_report(self, sessions):
        lines = ["login | time_in | time_out"]
        for session in sorted(sessions):
            lines.append(self.format_session_line(session))
        return "\n".join(lines) + "\n"

    def save_report(self, sessions):
        report_text = self.build_report(sessions)
        try:
            self._report_path.write_text(report_text, encoding="utf-8")
            return True, f"Отчёт сохранён: {self._report_path.resolve()}"
        except OSError as error:
            return False, f"Не удалось сохранить отчёт: {error}"
```


## `components.py`

```python
from datetime import datetime
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QFont
from PyQt6.QtWidgets import (
    QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, QLabel, QLineEdit,
    QPushButton, QTableWidget, QTableWidgetItem, QMessageBox, QGroupBox,
    QFormLayout, QStatusBar, QHeaderView
)
from models import User
from services import AuthenticationService, SessionService, ReportService


class HeaderLabel(QLabel):
    def __init__(self, text):
        super().__init__(text)
        self.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.setFont(QFont("Arial", 17, QFont.Weight.Bold))
        self.setObjectName("mainHeader")

    def setText(self, text):
        super().setText(str(text))


class HintLabel(QLabel):
    def __init__(self, text=""):
        super().__init__(text)
        self.setAlignment(Qt.AlignmentFlag.AlignLeft | Qt.AlignmentFlag.AlignVCenter)
        self.setWordWrap(True)

    def mousePressEvent(self, event):
        self.setFocus()
        super().mousePressEvent(event)


class UsernameInput(QLineEdit):
    def __init__(self):
        super().__init__()
        self.setPlaceholderText("Имя пользователя")
        self.setClearButtonEnabled(True)
        self.setMinimumHeight(34)

    def focusInEvent(self, event):
        self.setStyleSheet("border: 2px solid #3978c5; padding: 5px;")
        super().focusInEvent(event)

    def focusOutEvent(self, event):
        self.setStyleSheet("padding: 5px;")
        super().focusOutEvent(event)


class PasswordInput(QLineEdit):
    def __init__(self):
        super().__init__()
        self.setPlaceholderText("Пароль")
        self.setEchoMode(QLineEdit.EchoMode.Password)
        self.setMinimumHeight(34)

    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Return:
            self.returnPressed.emit()
            event.accept()
            return
        super().keyPressEvent(event)


class LoginButton(QPushButton):
    def __init__(self, text="Войти"):
        super().__init__(text)
        self.setMinimumHeight(36)
        self.setCursor(Qt.CursorShape.PointingHandCursor)

    def enterEvent(self, event):
        self.setStyleSheet("background-color: #245fa5; color: white; padding: 7px;")
        super().enterEvent(event)

    def leaveEvent(self, event):
        self.setStyleSheet("background-color: #3978c5; color: white; padding: 7px;")
        super().leaveEvent(event)


class LogoutButton(QPushButton):
    def __init__(self, text="Завершить сеанс"):
        super().__init__(text)
        self.setMinimumHeight(34)
        self.setCursor(Qt.CursorShape.PointingHandCursor)

    def mousePressEvent(self, event):
        if event.button() == Qt.MouseButton.LeftButton:
            self.setDown(True)
        super().mousePressEvent(event)


class RefreshButton(QPushButton):
    def __init__(self):
        super().__init__("Обновить журнал")
        self.setMinimumHeight(34)

    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Space:
            self.clicked.emit()
            event.accept()
            return
        super().keyPressEvent(event)


class SessionTable(QTableWidget):
    def __init__(self):
        super().__init__(0, 3)
        self.setHorizontalHeaderLabels(["Пользователь", "Время входа", "Время выхода"])
        self.setEditTriggers(QTableWidget.EditTrigger.NoEditTriggers)
        self.setSelectionBehavior(QTableWidget.SelectionBehavior.SelectRows)
        self.setAlternatingRowColors(True)
        self.horizontalHeader().setSectionResizeMode(QHeaderView.ResizeMode.Stretch)
        self.verticalHeader().setVisible(False)

    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Delete:
            self.clearSelection()
            event.accept()
            return
        super().keyPressEvent(event)

    def set_sessions(self, sessions):
        self.setRowCount(0)
        for session in sorted(sessions):
            row_index = self.rowCount()
            self.insertRow(row_index)
            values = [
                session.user.display_name,
                session.format_time_in(),
                session.format_time_out(),
            ]
            for column_index, value in enumerate(values):
                item = QTableWidgetItem(value)
                item.setTextAlignment(
                    Qt.AlignmentFlag.AlignLeft | Qt.AlignmentFlag.AlignVCenter
                )
                self.setItem(row_index, column_index, item)


class StatusLabel(QLabel):
    def __init__(self, text="Готово"):
        super().__init__(text)
        self.setAlignment(Qt.AlignmentFlag.AlignLeft | Qt.AlignmentFlag.AlignVCenter)
        self.setWordWrap(True)

    def changeEvent(self, event):
        super().changeEvent(event)
        if self.isEnabled():
            self.setStyleSheet("color: #245a35;")
        else:
            self.setStyleSheet("color: #777777;")


class ReportPathLabel(QLabel):
    def __init__(self, text):
        super().__init__(text)
        self.setAlignment(Qt.AlignmentFlag.AlignLeft | Qt.AlignmentFlag.AlignVCenter)
        self.setWordWrap(True)
        self.setTextInteractionFlags(Qt.TextInteractionFlag.TextSelectableByMouse)

    def resizeEvent(self, event):
        super().resizeEvent(event)
        self.setToolTip(self.text())


class AttendanceMainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self._current_user = None
        self._authentication_service = AuthenticationService(self._create_initial_users())
        self._session_service = SessionService()
        self._report_service = ReportService()
        self._login_time = None

        self.setWindowTitle("Система учёта посещений")
        self.setMinimumSize(850, 560)
        self._build_interface()
        self._apply_styles()
        self._refresh_table()

    @property
    def current_user(self):
        return self._current_user

    @current_user.setter
    def current_user(self, value):
        if value is not None and not isinstance(value, User):
            raise TypeError("Текущий пользователь должен быть объектом User или None.")
        self._current_user = value

    @staticmethod
    def _create_initial_users():
        # Учебные записи позволяют проверить журнал без предварительного заполнения базы.
        return [
            User("student01", "study01", "Алексей Иванов"),
            User("student02", "study02", "Мария Смирнова"),
            User("student03", "study03", "Иван Петров"),
            User("student04", "study04", "Анна Соколова"),
            User("student05", "study05", "Дмитрий Кузнецов"),
            User("student06", "study06", "Елена Попова"),
            User("student07", "study07", "Максим Васильев"),
            User("student08", "study08", "Ольга Павлова"),
            User("student09", "study09", "Никита Семёнов"),
            User("admin", "admin123", "Администратор", "Администратор"),
        ]

    def _build_interface(self):
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        root_layout = QVBoxLayout(central_widget)
        root_layout.setContentsMargins(22, 18, 22, 18)
        root_layout.setSpacing(14)

        self.header_label = HeaderLabel("Система учёта посещений пользователей")
        root_layout.addWidget(self.header_label)

        self.login_group = QGroupBox("Авторизация")
        login_layout = QFormLayout(self.login_group)
        login_layout.setLabelAlignment(Qt.AlignmentFlag.AlignRight)
        login_layout.setFormAlignment(Qt.AlignmentFlag.AlignHCenter | Qt.AlignmentFlag.AlignTop)

        self.username_input = UsernameInput()
        self.password_input = PasswordInput()
        self.login_button = LoginButton()
        login_layout.addRow("Логин:", self.username_input)
        login_layout.addRow("Пароль:", self.password_input)
        login_layout.addRow("", self.login_button)
        root_layout.addWidget(self.login_group)

        action_layout = QHBoxLayout()
        self.current_user_label = HintLabel("Текущий пользователь: не выполнен вход")
        self.logout_button = LogoutButton()
        self.refresh_button = RefreshButton()
        action_layout.addWidget(self.current_user_label, 1)
        action_layout.addWidget(self.refresh_button)
        action_layout.addWidget(self.logout_button)
        root_layout.addLayout(action_layout)

        self.session_table = SessionTable()
        root_layout.addWidget(self.session_table, 1)

        self.status_label = StatusLabel("Введите учётные данные для начала работы.")
        root_layout.addWidget(self.status_label)

        self.report_path_label = ReportPathLabel(
            f"Файл отчёта при закрытии: {self._report_service.report_path}"
        )
        root_layout.addWidget(self.report_path_label)

        self.setStatusBar(QStatusBar())
        self.statusBar().showMessage("Готово")

        self.login_button.clicked.connect(self._handle_login)
        self.password_input.returnPressed.connect(self._handle_login)
        self.logout_button.clicked.connect(self._handle_logout)
        self.refresh_button.clicked.connect(self._refresh_table)
        self.logout_button.setEnabled(False)

    def _apply_styles(self):
        self.setStyleSheet("""
            QMainWindow { background: #f4f6f8; }
            QGroupBox { font-weight: bold; border: 1px solid #c9d1d9;
                        border-radius: 7px; margin-top: 10px; padding: 10px; }
            QGroupBox::title { subcontrol-origin: margin; left: 10px; padding: 0 4px; }
            QLineEdit { border: 1px solid #aeb8c2; border-radius: 4px; padding: 5px; }
            QPushButton { background: #3978c5; color: white; border: none;
                          border-radius: 4px; padding: 7px 12px; }
            QPushButton:disabled { background: #aeb8c2; color: #f4f4f4; }
            QTableWidget { background: white; border: 1px solid #d2d8de; }
            QHeaderView::section { background: #e5ebf1; padding: 7px; border: none; }
        """)
        self.login_button.setStyleSheet("background-color: #3978c5; color: white; padding: 7px;")

    def _handle_login(self):
        username = self.username_input.text().strip()
        password = self.password_input.text()

        if not username or not password:
            self._show_status("Заполните оба поля перед авторизацией.", error=True)
            return

        authenticated_user = self._authentication_service.authenticate(username, password)
        if authenticated_user is None:
            self._show_status("Неверный логин или пароль.", error=True)
            self.password_input.clear()
            return

        if self._current_user is not None:
            self._session_service.finish_session(self._current_user.username)

        self.current_user = authenticated_user
        self._login_time = datetime.now()
        self._session_service.start_session(authenticated_user)
        self.current_user_label.setText(
            f"Текущий пользователь: {authenticated_user.display_name} "
            f"({authenticated_user.username})"
        )
        self.logout_button.setEnabled(True)
        self.username_input.setEnabled(False)
        self.password_input.clear()
        self._show_status(f"Вход выполнен: {self._login_time.strftime('%d.%m.%Y %H:%M:%S')}")
        self._refresh_table()

    def _handle_logout(self):
        if self._current_user is None:
            self._show_status("Нет активного сеанса для завершения.", error=True)
            return

        username = self._current_user.username
        closed_session = self._session_service.finish_session(username)
        self.current_user = None
        self._login_time = None
        self.current_user_label.setText("Текущий пользователь: не выполнен вход")
        self.logout_button.setEnabled(False)
        self.username_input.setEnabled(True)
        self._show_status(f"Сеанс пользователя {username} завершён.")
        self._refresh_table()

        if closed_session is None:
            self.statusBar().showMessage("Активный сеанс не найден.", 5000)

    def _refresh_table(self):
        self.session_table.set_sessions(self._session_service.sessions)
        self.statusBar().showMessage(
            f"Записей в журнале: {len(self._session_service.sessions)}"
        )

    def _show_status(self, message, error=False):
        self.status_label.setText(message)
        if error:
            self.status_label.setStyleSheet("color: #a12622;")
        else:
            self.status_label.setStyleSheet("color: #245a35;")
        self.statusBar().showMessage(message, 6000)

    def closeEvent(self, event):
        # Закрытие окна фиксирует выход, чтобы в отчёте не оставались открытые сеансы.
        self._session_service.close_all_active_sessions()
        success, message = self._report_service.save_report(self._session_service.sessions)
        if success:
            self.statusBar().showMessage(message)
            event.accept()
        else:
            QMessageBox.warning(self, "Ошибка сохранения отчёта", message)
            # Закрытие не блокируется: пользователь получает предупреждение,
            # а приложение не зависает из-за недоступного файла.
            event.accept()
```


## `requirements.txt`

```text
PyQt6>=6.6.0
```


## `README.md`

```markdown
# Система учёта посещений пользователей

Учебный проект на Python и PyQt6: авторизация, регистрация времени входа/выхода,
табличный журнал и сохранение отчёта при закрытии окна.

## Требования

- Python 3.10 или новее
- PyQt6

## Установка и запуск

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

macOS/Linux:

```bash
source .venv/bin/activate
```

Затем:

```bash
pip install -r requirements.txt
python main.py
```

## Учебные учётные записи

- `admin` / `admin123`
- `student01` / `study01`
- `student02` / `study02`
- далее `student03`–`student09` с паролями `study03`–`study09`

Пароли в открытом виде используются только для демонстрации учебного проекта.
В реальном приложении их следует хранить в виде стойких хешей.

## Отчёт

При закрытии окна создаётся `attendance_report.txt` в текущей рабочей папке.
Формат строк: `login | time_in | time_out`.
Дата и время записываются в формате `%d.%m.%Y %H:%M:%S`.

## Состав проекта

- `main.py` — запуск приложения.
- `models.py` — абстрактная сущность, пользователи, сессии и собственный маппинг.
- `services.py` — авторизация, управление сессиями и запись отчёта.
- `components.py` — переопределённые UI-компоненты и главное окно.
- `requirements.txt` — зависимость PyQt6.
```
