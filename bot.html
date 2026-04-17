"""
╔══════════════════════════════════════════════════════╗
║           BALBES TEAM — Telegram Bot                 ║
║   Aiogram 3.x | SQLite (синхронный) | Pydroid3 OK   ║
╚══════════════════════════════════════════════════════╝

Установка зависимостей (ТОЛЬКО aiogram, aiosqlite НЕ нужен):
    pip install aiogram==3.7.0

Запуск:
    python bot.py
"""

import asyncio
import logging
import os
import sqlite3                          # стандартная библиотека, pip не нужен
from aiogram import Bot, Dispatcher, F, Router
from aiogram.enums import ParseMode
from aiogram.filters import Command, CommandStart
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage
from aiogram.types import (
    BotCommand, CallbackQuery, Document, InlineKeyboardButton,
    InlineKeyboardMarkup, Message, ReplyKeyboardMarkup, KeyboardButton,
    ReplyKeyboardRemove
)

# ─── КОНФИГ ───────────────────────────────────────────────────────────────────
BOT_TOKEN   = os.getenv("TOKEN", "YOUR_BOT_TOKEN")
ADMIN_IDS   = [int(x) for x in os.getenv("ADMIN_IDS", "YOUR_ADMIN_ID").split(",") if x.strip().lstrip("-").isdigit()]
DB_PATH     = "BAlbes.db"
ALLOWED_EXT = {".py", ".html", ".css", ".js", ".zip", ".rar", ".txt"}

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
log = logging.getLogger(__name__)

# ─── FSM СОСТОЯНИЯ ────────────────────────────────────────────────────────────
class ProjectStates(StatesGroup):
    waiting_name       = State()
    waiting_desc       = State()
    waiting_edit_field = State()
    waiting_edit_value = State()

class RoleStates(StatesGroup):
    waiting_name       = State()
    waiting_project_id = State()
    waiting_stage      = State()
    assign_user_id     = State()
    assign_role_id     = State()
    revoke_user_id     = State()
    revoke_role_id     = State()

class TaskStates(StatesGroup):
    waiting_user_id    = State()
    waiting_text       = State()
    waiting_project_id = State()
    waiting_reject_msg = State()
    sending_files_task = State()

class UserStates(StatesGroup):
    waiting_time_uid   = State()
    waiting_time_delta = State()

# ─── БАЗА ДАННЫХ (синхронная, sqlite3) ────────────────────────────────────────
def get_db() -> sqlite3.Connection:
    """Открывает соединение с БД. После использования закрывать conn.close()."""
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row          # доступ row["field"]
    conn.execute("PRAGMA foreign_keys = ON")
    return conn

def init_db():
    """Создаёт все таблицы при первом запуске. Синхронная, вызывается без await."""
    conn = get_db()
    conn.executescript("""
        CREATE TABLE IF NOT EXISTS users (
            id          INTEGER PRIMARY KEY,
            username    TEXT,
            full_name   TEXT,
            joined_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        CREATE TABLE IF NOT EXISTS projects (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            name        TEXT NOT NULL,
            description TEXT,
            announce    TEXT DEFAULT 'Скоро задачи отправятся!',
            created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        CREATE TABLE IF NOT EXISTS roles (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            name        TEXT NOT NULL,
            project_id  INTEGER REFERENCES projects(id) ON DELETE CASCADE,
            stage       TEXT NOT NULL
        );
        CREATE TABLE IF NOT EXISTS user_roles (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id     INTEGER REFERENCES users(id) ON DELETE CASCADE,
            role_id     INTEGER REFERENCES roles(id) ON DELETE CASCADE,
            assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            UNIQUE(user_id, role_id)
        );
        CREATE TABLE IF NOT EXISTS tasks (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id     INTEGER REFERENCES users(id) ON DELETE CASCADE,
            project_id  INTEGER REFERENCES projects(id) ON DELETE SET NULL,
            text        TEXT NOT NULL,
            status      TEXT DEFAULT 'active',
            reject_msg  TEXT,
            created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            closed_at   TIMESTAMP
        );
        CREATE TABLE IF NOT EXISTS task_files (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            task_id     INTEGER REFERENCES tasks(id) ON DELETE CASCADE,
            file_id     TEXT NOT NULL,
            file_name   TEXT,
            uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        CREATE TABLE IF NOT EXISTS user_projects_time (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id     INTEGER REFERENCES users(id) ON DELETE CASCADE,
            project_id  INTEGER REFERENCES projects(id) ON DELETE CASCADE,
            hours       REAL DEFAULT 0
        );
    """)
    conn.commit()
    conn.close()
    log.info("✅ База данных инициализирована: %s", DB_PATH)

# ─── ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ─────────────────────────────────────────────────
def is_admin(user_id: int) -> bool:
    return user_id in ADMIN_IDS

def ik(*rows) -> InlineKeyboardMarkup:
    """Быстрое создание InlineKeyboard из списков (text, callback_data)."""
    return InlineKeyboardMarkup(
        inline_keyboard=[[InlineKeyboardButton(text=t, callback_data=d) for t, d in row] for row in rows]
    )

def ensure_user(user):
    """Регистрирует пользователя при первом обращении (синхронно)."""
    conn = get_db()
    conn.execute(
        "INSERT OR IGNORE INTO users(id, username, full_name) VALUES(?,?,?)",
        (user.id, user.username, user.full_name)
    )
    conn.commit()
    conn.close()

async def send_admin_panel(target, state: FSMContext = None):
    """Отправляет/редактирует главное меню админки."""
    if state:
        await state.clear()
    kb = ik(
        [("📁 Проекты", "adm:projects"), ("🏷 Роли", "adm:roles")],
        [("📋 Задачи",  "adm:tasks"),    ("👥 Пользователи", "adm:users")],
        [("📎 Файлы задач", "adm:files")],
    )
    text = "🛠 <b>Панель BALBES TEAM</b>\nВыберите раздел:"
    if isinstance(target, CallbackQuery):
        await target.message.edit_text(text, reply_markup=kb)
    else:
        await target.answer(text, reply_markup=kb)

# ─── РОУТЕР ───────────────────────────────────────────────────────────────────
router = Router()

# ═══════════════════════════════════════════════════════════════════════════════
#  /start
# ═══════════════════════════════════════════════════════════════════════════════
@router.message(CommandStart())
async def cmd_start(msg: Message):
    ensure_user(msg.from_user)
    kb = ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="👤 Мой профиль"), KeyboardButton(text="📋 Мои задачи")]],
        resize_keyboard=True
    )
    await msg.answer(
        "👋 Добро пожаловать в <b>BALBES TEAM</b>!\n\n"
        "Здесь ты можешь отслеживать свои задачи, роли и прогресс в командных проектах.\n\n"
        "Используй кнопки ниже 👇",
        reply_markup=kb
    )

# ═══════════════════════════════════════════════════════════════════════════════
#  МОЙ ПРОФИЛЬ
# ═══════════════════════════════════════════════════════════════════════════════
@router.message(F.text == "👤 Мой профиль")
async def my_profile(msg: Message):
    uid = msg.from_user.id
    ensure_user(msg.from_user)

    conn = get_db()
    roles_rows = conn.execute("""
        SELECT r.name, r.stage, p.name AS pname
        FROM user_roles ur
        JOIN roles r ON r.id = ur.role_id
        JOIN projects p ON p.id = r.project_id
        WHERE ur.user_id = ?
    """, (uid,)).fetchall()
    proj_count = conn.execute("""
        SELECT COUNT(DISTINCT r.project_id)
        FROM user_roles ur JOIN roles r ON r.id = ur.role_id
        WHERE ur.user_id = ?
    """, (uid,)).fetchone()
    total_time = conn.execute(
        "SELECT COALESCE(SUM(hours),0) FROM user_projects_time WHERE user_id=?", (uid,)
    ).fetchone()
    done_tasks = conn.execute("""
        SELECT t.text, p.name AS pname FROM tasks t
        LEFT JOIN projects p ON p.id = t.project_id
        WHERE t.user_id=? AND t.status='done'
    """, (uid,)).fetchall()
    conn.close()

    roles_txt = "\n".join(
        f"  • <b>{r['name']}</b> [{r['stage']}] — {r['pname']}" for r in roles_rows
    ) or "  — нет ролей"
    done_txt = "\n".join(
        f"  ✅ {t['text'][:60]} <i>({t['pname'] or '—'})</i>" for t in done_tasks
    ) or "  — нет выполненных задач"

    await msg.answer(
        f"👤 <b>Профиль: {msg.from_user.full_name}</b>\n"
        f"🆔 ID: <code>{uid}</code>\n\n"
        f"🏷 <b>Роли:</b>\n{roles_txt}\n\n"
        f"📁 <b>Проектов:</b> {proj_count[0]}\n"
        f"⏱ <b>Время в тиме:</b> {total_time[0]:.1f} ч.\n\n"
        f"✅ <b>Выполненные задачи:</b>\n{done_txt}"
    )

# ═══════════════════════════════════════════════════════════════════════════════
#  МОИ ЗАДАЧИ
# ═══════════════════════════════════════════════════════════════════════════════
@router.message(F.text == "📋 Мои задачи")
async def my_tasks(msg: Message):
    uid = msg.from_user.id
    ensure_user(msg.from_user)

    conn = get_db()
    tasks = conn.execute("""
        SELECT t.id, t.text, t.status, p.name AS pname
        FROM tasks t LEFT JOIN projects p ON p.id = t.project_id
        WHERE t.user_id=? AND t.status IN ('active','rejected')
        ORDER BY t.created_at DESC
    """, (uid,)).fetchall()
    conn.close()

    if not tasks:
        await msg.answer("📭 У тебя нет активных задач. Ожидай назначений от администратора!")
        return

    for t in tasks:
        status_icon = "🔴 Отклонена" if t["status"] == "rejected" else "🟡 Активна"
        kb = ik([("📎 Отправить файлы", f"send_files:{t['id']}")])
        await msg.answer(
            f"<b>Задача #{t['id']}</b> [{status_icon}]\n"
            f"📁 Проект: {t['pname'] or '—'}\n\n"
            f"{t['text']}",
            reply_markup=kb
        )

# ═══════════════════════════════════════════════════════════════════════════════
#  ОТПРАВКА ФАЙЛОВ (пользователь)
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data.startswith("send_files:"))
async def cb_send_files_start(call: CallbackQuery, state: FSMContext):
    task_id = int(call.data.split(":")[1])

    conn = get_db()
    task = conn.execute(
        "SELECT * FROM tasks WHERE id=? AND user_id=?", (task_id, call.from_user.id)
    ).fetchone()
    conn.close()

    if not task:
        await call.answer("Задача не найдена!", show_alert=True)
        return

    await state.set_state(TaskStates.sending_files_task)
    await state.update_data(task_id=task_id)
    await call.message.answer(
        f"📎 Отправь файлы по задаче <b>#{task_id}</b>.\n"
        f"Разрешённые форматы: <code>.py .html .css .js .zip .rar .txt</code>\n\n"
        f"Когда отправишь все файлы, нажми кнопку ниже.",
        reply_markup=ik([("✅ Готово, файлы отправлены", f"files_done:{task_id}")])
    )
    await call.answer()

@router.message(TaskStates.sending_files_task, F.document)
async def receive_task_file(msg: Message, state: FSMContext, bot: Bot):
    data    = await state.get_data()
    task_id = data.get("task_id")
    doc: Document = msg.document
    ext = os.path.splitext(doc.file_name or "")[1].lower()

    if ext not in ALLOWED_EXT:
        await msg.answer(f"⛔ Файл <b>{doc.file_name}</b> имеет недопустимое расширение. Пропущен.")
        return

    conn = get_db()
    conn.execute(
        "INSERT INTO task_files(task_id, file_id, file_name) VALUES(?,?,?)",
        (task_id, doc.file_id, doc.file_name)
    )
    conn.commit()
    conn.close()

    await msg.answer(f"✅ Файл <b>{doc.file_name}</b> принят!")

@router.callback_query(F.data.startswith("files_done:"))
async def cb_files_done(call: CallbackQuery, state: FSMContext, bot: Bot):
    task_id = int(call.data.split(":")[1])

    conn = get_db()
    conn.execute(
        "UPDATE tasks SET status='pending_review' WHERE id=? AND user_id=?",
        (task_id, call.from_user.id)
    )
    conn.commit()
    files_count = conn.execute(
        "SELECT COUNT(*) FROM task_files WHERE task_id=?", (task_id,)
    ).fetchone()
    conn.close()

    await state.clear()
    await call.message.edit_reply_markup()
    await call.message.answer(
        f"📬 Задача <b>#{task_id}</b> отправлена на проверку!\n"
        f"Прикреплено файлов: {files_count[0]}\n\nОжидай обратной связи от администратора."
    )
    for adm_id in ADMIN_IDS:
        try:
            await bot.send_message(
                adm_id,
                f"🔔 <b>Задача #{task_id}</b> ожидает проверки!\n"
                f"👤 От: {call.from_user.full_name} (<code>{call.from_user.id}</code>)\n"
                f"📎 Файлов: {files_count[0]}"
            )
        except Exception:
            pass
    await call.answer()

# ═══════════════════════════════════════════════════════════════════════════════
#  /admin
# ═══════════════════════════════════════════════════════════════════════════════
@router.message(Command("admin"))
async def cmd_admin(msg: Message, state: FSMContext):
    if not is_admin(msg.from_user.id):
        await msg.answer("⛔ Нет доступа.")
        return
    await send_admin_panel(msg, state)

@router.callback_query(F.data == "adm:main")
async def cb_adm_main(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id):
        await call.answer("⛔ Нет доступа.")
        return
    await send_admin_panel(call, state)
    await call.answer()

# ═══════════════════════════════════════════════════════════════════════════════
#  АДМИН — ПРОЕКТЫ
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data == "adm:projects")
async def cb_adm_projects(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    kb = ik(
        [("➕ Создать проект", "adm:proj_create")],
        [("📋 Список проектов", "adm:proj_list")],
        [("🔙 Назад", "adm:main")],
    )
    await call.message.edit_text("📁 <b>Проекты</b>", reply_markup=kb)
    await call.answer()

@router.callback_query(F.data == "adm:proj_create")
async def cb_proj_create(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(ProjectStates.waiting_name)
    await call.message.edit_text("📁 Введите <b>название</b> нового проекта:")
    await call.answer()

@router.message(ProjectStates.waiting_name)
async def proj_name_received(msg: Message, state: FSMContext):
    await state.update_data(name=msg.text.strip())
    await state.set_state(ProjectStates.waiting_desc)
    await msg.answer("📝 Введите <b>описание</b> проекта:")

@router.message(ProjectStates.waiting_desc)
async def proj_desc_received(msg: Message, state: FSMContext):
    data = await state.get_data()

    conn = get_db()
    conn.execute(
        "INSERT INTO projects(name, description) VALUES(?,?)",
        (data["name"], msg.text.strip())
    )
    conn.commit()
    conn.close()

    await state.clear()
    await msg.answer(
        f"✅ Проект <b>{data['name']}</b> создан!\n"
        f"Сообщение по умолчанию: «Скоро задачи отправятся!»",
        reply_markup=ik([("🔙 В панель", "adm:main")])
    )

@router.callback_query(F.data == "adm:proj_list")
async def cb_proj_list(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return

    conn     = get_db()
    projects = conn.execute("SELECT * FROM projects ORDER BY id DESC").fetchall()
    conn.close()

    if not projects:
        await call.message.edit_text("📂 Проектов нет.", reply_markup=ik([("🔙 Назад", "adm:projects")]))
        await call.answer()
        return

    rows = [[(f"📁 {p['name']}", f"adm:proj_view:{p['id']}")] for p in projects]
    rows.append([("🔙 Назад", "adm:projects")])
    await call.message.edit_text("📋 <b>Все проекты:</b>", reply_markup=ik(*rows))
    await call.answer()

@router.callback_query(F.data.startswith("adm:proj_view:"))
async def cb_proj_view(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    pid = int(call.data.split(":")[2])

    conn = get_db()
    p    = conn.execute("SELECT * FROM projects WHERE id=?", (pid,)).fetchone()
    conn.close()

    if not p:
        await call.message.edit_text("Проект не найден.", reply_markup=ik([("🔙 Назад", "adm:proj_list")]))
        await call.answer()
        return

    text = (
        f"📁 <b>{p['name']}</b>\n"
        f"📝 {p['description'] or '—'}\n"
        f"📣 Анонс: {p['announce']}\n"
        f"📅 Создан: {p['created_at']}"
    )
    kb = ik(
        [("✏️ Редактировать", f"adm:proj_edit:{pid}"), ("🗑 Удалить", f"adm:proj_del:{pid}")],
        [("🔙 Назад", "adm:proj_list")]
    )
    await call.message.edit_text(text, reply_markup=kb)
    await call.answer()

@router.callback_query(F.data.startswith("adm:proj_del:"))
async def cb_proj_del(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    pid = int(call.data.split(":")[2])

    conn = get_db()
    conn.execute("DELETE FROM projects WHERE id=?", (pid,))
    conn.commit()
    conn.close()

    await call.message.edit_text("🗑 Проект удалён.", reply_markup=ik([("🔙 К проектам", "adm:projects")]))
    await call.answer()

@router.callback_query(F.data.startswith("adm:proj_edit:"))
async def cb_proj_edit(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    pid = int(call.data.split(":")[2])
    await state.update_data(edit_proj_id=pid)
    await state.set_state(ProjectStates.waiting_edit_field)
    kb = ik(
        [("Название", f"proj_ef:name:{pid}"), ("Описание", f"proj_ef:description:{pid}")],
        [("Анонс", f"proj_ef:announce:{pid}")],
        [("🔙 Отмена", f"adm:proj_view:{pid}")]
    )
    await call.message.edit_text("✏️ Что редактируем?", reply_markup=kb)
    await call.answer()

@router.callback_query(F.data.startswith("proj_ef:"))
async def cb_proj_edit_field(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    _, field, pid = call.data.split(":")
    await state.update_data(edit_field=field, edit_proj_id=int(pid))
    await state.set_state(ProjectStates.waiting_edit_value)
    await call.message.edit_text(f"✏️ Введите новое значение для <b>{field}</b>:")
    await call.answer()

@router.message(ProjectStates.waiting_edit_value)
async def proj_edit_value(msg: Message, state: FSMContext):
    data = await state.get_data()
    pid, field, val = data["edit_proj_id"], data["edit_field"], msg.text.strip()

    conn = get_db()
    conn.execute(f"UPDATE projects SET {field}=? WHERE id=?", (val, pid))
    conn.commit()
    conn.close()

    await state.clear()
    await msg.answer("✅ Обновлено!", reply_markup=ik([("🔙 К проектам", "adm:proj_list")]))

# ═══════════════════════════════════════════════════════════════════════════════
#  АДМИН — РОЛИ
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data == "adm:roles")
async def cb_adm_roles(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    kb = ik(
        [("➕ Создать роль",    "adm:role_create")],
        [("👤 Назначить роль", "adm:role_assign")],
        [("❌ Снять роль",     "adm:role_revoke")],
        [("🔙 Назад", "adm:main")],
    )
    await call.message.edit_text("🏷 <b>Роли</b>", reply_markup=kb)
    await call.answer()

@router.callback_query(F.data == "adm:role_create")
async def cb_role_create(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(RoleStates.waiting_name)
    await call.message.edit_text("🏷 Введите <b>название</b> роли (например: Frontend Developer):")
    await call.answer()

@router.message(RoleStates.waiting_name)
async def role_name(msg: Message, state: FSMContext):
    await state.update_data(role_name=msg.text.strip())

    conn     = get_db()
    projects = conn.execute("SELECT id, name FROM projects ORDER BY id").fetchall()
    conn.close()

    if not projects:
        await msg.answer("⚠️ Сначала создайте хотя бы один проект!", reply_markup=ik([("🔙 В панель", "adm:main")]))
        await state.clear()
        return

    rows = [[(p["name"], f"role_proj:{p['id']}")] for p in projects]
    await msg.answer("📁 К какому проекту привязать роль?", reply_markup=ik(*rows))
    await state.set_state(RoleStates.waiting_project_id)

@router.callback_query(RoleStates.waiting_project_id, F.data.startswith("role_proj:"))
async def role_project(call: CallbackQuery, state: FSMContext):
    pid = int(call.data.split(":")[1])
    await state.update_data(role_project_id=pid)
    await state.set_state(RoleStates.waiting_stage)
    stages = ["Frontend", "Backend", "QA", "Team Lead", "Design", "DevOps", "Marketing"]
    rows   = [[(s, f"role_stage:{s}")] for s in stages]
    await call.message.edit_text("🔧 Выберите этап разработки:", reply_markup=ik(*rows))
    await call.answer()

@router.callback_query(RoleStates.waiting_stage, F.data.startswith("role_stage:"))
async def role_stage(call: CallbackQuery, state: FSMContext):
    stage = call.data.split(":")[1]
    data  = await state.get_data()

    conn = get_db()
    conn.execute(
        "INSERT INTO roles(name, project_id, stage) VALUES(?,?,?)",
        (data["role_name"], data["role_project_id"], stage)
    )
    conn.commit()
    conn.close()

    await state.clear()
    await call.message.edit_text(
        f"✅ Роль <b>{data['role_name']}</b> [{stage}] создана!",
        reply_markup=ik([("🔙 К ролям", "adm:roles")])
    )
    await call.answer()

@router.callback_query(F.data == "adm:role_assign")
async def cb_role_assign_start(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(RoleStates.assign_user_id)
    await call.message.edit_text("👤 Введите <b>Telegram ID</b> пользователя для назначения роли:")
    await call.answer()

@router.message(RoleStates.assign_user_id)
async def role_assign_user(msg: Message, state: FSMContext):
    if not msg.text.strip().lstrip("-").isdigit():
        await msg.answer("⚠️ Введите числовой ID.")
        return
    uid = int(msg.text.strip())

    conn  = get_db()
    user  = conn.execute("SELECT * FROM users WHERE id=?", (uid,)).fetchone()
    roles = conn.execute("""
        SELECT r.id, r.name, r.stage, p.name AS pname FROM roles r
        JOIN projects p ON p.id = r.project_id ORDER BY r.id
    """).fetchall()
    conn.close()

    if not user:
        await msg.answer("❌ Пользователь не найден в БД. Он должен был запустить бота (/start).")
        return
    if not roles:
        await msg.answer("⚠️ Нет ролей. Создайте роли.", reply_markup=ik([("🔙 К ролям", "adm:roles")]))
        await state.clear()
        return

    await state.update_data(assign_uid=uid)
    rows = [[(f"{r['name']} [{r['stage']}] — {r['pname']}", f"asgn_role:{r['id']}")] for r in roles]
    await msg.answer(f"🏷 Выберите роль для пользователя <code>{uid}</code>:", reply_markup=ik(*rows))
    await state.set_state(RoleStates.assign_role_id)

@router.callback_query(RoleStates.assign_role_id, F.data.startswith("asgn_role:"))
async def role_assign_confirm(call: CallbackQuery, state: FSMContext):
    rid  = int(call.data.split(":")[1])
    data = await state.get_data()
    uid  = data["assign_uid"]

    conn = get_db()
    try:
        conn.execute("INSERT INTO user_roles(user_id, role_id) VALUES(?,?)", (uid, rid))
        conn.commit()
        await call.message.edit_text("✅ Роль назначена!", reply_markup=ik([("🔙 К ролям", "adm:roles")]))
    except sqlite3.IntegrityError:
        await call.message.edit_text("ℹ️ У пользователя уже есть эта роль.", reply_markup=ik([("🔙 К ролям", "adm:roles")]))
    finally:
        conn.close()

    await state.clear()
    await call.answer()

@router.callback_query(F.data == "adm:role_revoke")
async def cb_role_revoke(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(RoleStates.revoke_user_id)
    await call.message.edit_text("👤 Введите <b>Telegram ID</b> пользователя для снятия роли:")
    await call.answer()

@router.message(RoleStates.revoke_user_id)
async def role_revoke_uid(msg: Message, state: FSMContext):
    if not msg.text.strip().lstrip("-").isdigit():
        await msg.answer("⚠️ Введите числовой ID.")
        return
    uid = int(msg.text.strip())
    await state.update_data(revoke_uid=uid)

    conn  = get_db()
    roles = conn.execute("""
        SELECT r.id, r.name, r.stage, p.name AS pname
        FROM user_roles ur JOIN roles r ON r.id=ur.role_id
        JOIN projects p ON p.id=r.project_id
        WHERE ur.user_id=?
    """, (uid,)).fetchall()
    conn.close()

    if not roles:
        await msg.answer("У пользователя нет ролей.", reply_markup=ik([("🔙 К ролям", "adm:roles")]))
        await state.clear()
        return

    rows = [[(f"{r['name']} [{r['stage']}] — {r['pname']}", f"rev_role:{r['id']}")] for r in roles]
    await msg.answer("🏷 Выберите роль для снятия:", reply_markup=ik(*rows))
    await state.set_state(RoleStates.revoke_role_id)

@router.callback_query(RoleStates.revoke_role_id, F.data.startswith("rev_role:"))
async def role_revoke_confirm(call: CallbackQuery, state: FSMContext):
    rid  = int(call.data.split(":")[1])
    data = await state.get_data()

    conn = get_db()
    conn.execute("DELETE FROM user_roles WHERE user_id=? AND role_id=?", (data["revoke_uid"], rid))
    conn.commit()
    conn.close()

    await state.clear()
    await call.message.edit_text("✅ Роль снята!", reply_markup=ik([("🔙 К ролям", "adm:roles")]))
    await call.answer()

# ═══════════════════════════════════════════════════════════════════════════════
#  АДМИН — ЗАДАЧИ
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data == "adm:tasks")
async def cb_adm_tasks(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    kb = ik(
        [("➕ Назначить задачу", "adm:task_create")],
        [("🟡 Активные задачи",  "adm:tasks_active")],
        [("🔍 На проверке",      "adm:tasks_review")],
        [("✅ Выполненные",      "adm:tasks_done")],
        [("🔙 Назад", "adm:main")],
    )
    await call.message.edit_text("📋 <b>Задачи</b>", reply_markup=kb)
    await call.answer()

@router.callback_query(F.data == "adm:task_create")
async def cb_task_create(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(TaskStates.waiting_user_id)
    await call.message.edit_text("📋 Введите <b>Telegram ID</b> пользователя для задачи:")
    await call.answer()

@router.message(TaskStates.waiting_user_id)
async def task_uid(msg: Message, state: FSMContext):
    if not msg.text.strip().lstrip("-").isdigit():
        await msg.answer("⚠️ Введите числовой ID.")
        return
    uid = int(msg.text.strip())

    conn = get_db()
    user = conn.execute("SELECT * FROM users WHERE id=?", (uid,)).fetchone()
    conn.close()

    if not user:
        await msg.answer("❌ Пользователь не найден в БД.")
        return

    await state.update_data(task_uid=uid, task_uname=user["full_name"])
    await state.set_state(TaskStates.waiting_text)
    await msg.answer(f"✍️ Введите текст задачи для <b>{user['full_name']}</b>:")

@router.message(TaskStates.waiting_text)
async def task_text_handler(msg: Message, state: FSMContext):
    await state.update_data(task_text=msg.text.strip())

    conn     = get_db()
    projects = conn.execute("SELECT id, name FROM projects ORDER BY id").fetchall()
    conn.close()

    if not projects:
        await msg.answer("⚠️ Нет проектов. Создайте проект.", reply_markup=ik([("🔙 В панель", "adm:main")]))
        await state.clear()
        return

    rows = [[(p["name"], f"task_proj:{p['id']}")] for p in projects]
    rows.append([("— Без проекта —", "task_proj:0")])
    await msg.answer("📁 К какому проекту привязать задачу?", reply_markup=ik(*rows))
    await state.set_state(TaskStates.waiting_project_id)

@router.callback_query(TaskStates.waiting_project_id, F.data.startswith("task_proj:"))
async def task_project(call: CallbackQuery, state: FSMContext, bot: Bot):
    pid_raw = call.data.split(":")[1]
    pid     = int(pid_raw) if pid_raw != "0" else None
    data    = await state.get_data()

    conn   = get_db()
    cursor = conn.execute(
        "INSERT INTO tasks(user_id, project_id, text) VALUES(?,?,?)",
        (data["task_uid"], pid, data["task_text"])
    )
    task_id = cursor.lastrowid
    conn.commit()
    conn.close()

    await state.clear()
    await call.message.edit_text(
        f"✅ Задача <b>#{task_id}</b> назначена пользователю <code>{data['task_uid']}</code>!",
        reply_markup=ik([("🔙 К задачам", "adm:tasks")])
    )
    try:
        await bot.send_message(
            data["task_uid"],
            f"📬 <b>Тебе назначена новая задача!</b>\n\n{data['task_text']}\n\n"
            f"Используй кнопку «📋 Мои задачи» для просмотра и отправки файлов."
        )
    except Exception:
        pass
    await call.answer()

async def send_tasks_list(call: CallbackQuery, status: str, title: str):
    conn  = get_db()
    tasks = conn.execute("""
        SELECT t.id, t.text, t.user_id, u.full_name, p.name AS pname
        FROM tasks t
        LEFT JOIN users u ON u.id=t.user_id
        LEFT JOIN projects p ON p.id=t.project_id
        WHERE t.status=? ORDER BY t.created_at DESC
    """, (status,)).fetchall()
    conn.close()

    if not tasks:
        await call.message.edit_text(f"{title}\n\nПусто.", reply_markup=ik([("🔙 К задачам", "adm:tasks")]))
        await call.answer()
        return

    rows = [[(f"#{t['id']} {t['full_name'] or t['user_id']} — {(t['text'] or '')[:30]}…",
              f"adm:task_detail:{t['id']}")] for t in tasks]
    rows.append([("🔙 К задачам", "adm:tasks")])
    await call.message.edit_text(title, reply_markup=ik(*rows))
    await call.answer()

@router.callback_query(F.data == "adm:tasks_active")
async def cb_tasks_active(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await send_tasks_list(call, "active", "🟡 <b>Активные задачи:</b>")

@router.callback_query(F.data == "adm:tasks_review")
async def cb_tasks_review(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await send_tasks_list(call, "pending_review", "🔍 <b>Задачи на проверке:</b>")

@router.callback_query(F.data == "adm:tasks_done")
async def cb_tasks_done(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await send_tasks_list(call, "done", "✅ <b>Выполненные задачи:</b>")

@router.callback_query(F.data.startswith("adm:task_detail:"))
async def cb_task_detail(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    tid = int(call.data.split(":")[2])

    conn  = get_db()
    t     = conn.execute("""
        SELECT t.*, u.full_name, u.username, p.name AS pname
        FROM tasks t LEFT JOIN users u ON u.id=t.user_id
        LEFT JOIN projects p ON p.id=t.project_id WHERE t.id=?
    """, (tid,)).fetchone()
    files = conn.execute("SELECT * FROM task_files WHERE task_id=?", (tid,)).fetchall()
    conn.close()

    if not t:
        await call.message.edit_text("Задача не найдена.")
        await call.answer()
        return

    text = (
        f"📋 <b>Задача #{t['id']}</b> [{t['status']}]\n"
        f"👤 {t['full_name']} (@{t['username']} / <code>{t['user_id']}</code>)\n"
        f"📁 Проект: {t['pname'] or '—'}\n\n"
        f"{t['text']}\n\n"
        f"📎 Файлов: {len(files)}\n"
        + (f"💬 Отказ: {t['reject_msg']}" if t["reject_msg"] else "")
    )
    buttons = []
    if t["status"] == "pending_review":
        buttons.append([("✅ Принять задачу", f"adm:task_accept:{tid}"),
                        ("❌ Отклонить",      f"adm:task_reject:{tid}")])
    if files:
        buttons.append([("📎 Показать файлы", f"adm:task_files:{tid}")])
    buttons.append([("🔙 К задачам", "adm:tasks")])
    await call.message.edit_text(text, reply_markup=ik(*buttons))
    await call.answer()

@router.callback_query(F.data.startswith("adm:task_accept:"))
async def cb_task_accept(call: CallbackQuery, bot: Bot):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    tid = int(call.data.split(":")[2])

    conn = get_db()
    conn.execute("UPDATE tasks SET status='done', closed_at=CURRENT_TIMESTAMP WHERE id=?", (tid,))
    conn.commit()
    task = conn.execute("SELECT * FROM tasks WHERE id=?", (tid,)).fetchone()
    conn.close()

    await call.message.edit_text("✅ Задача принята и закрыта!", reply_markup=ik([("🔙 К задачам", "adm:tasks")]))
    try:
        await bot.send_message(task["user_id"], f"🎉 Задача <b>#{tid}</b> принята! Отличная работа!")
    except Exception:
        pass
    await call.answer()

@router.callback_query(F.data.startswith("adm:task_reject:"))
async def cb_task_reject_start(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    tid = int(call.data.split(":")[2])
    await state.set_state(TaskStates.waiting_reject_msg)
    await state.update_data(reject_tid=tid)
    await call.message.edit_text(f"💬 Введите <b>причину отклонения</b> задачи #{tid}:")
    await call.answer()

@router.message(TaskStates.waiting_reject_msg)
async def task_reject_msg(msg: Message, state: FSMContext, bot: Bot):
    data   = await state.get_data()
    tid    = data["reject_tid"]
    reason = msg.text.strip()

    conn = get_db()
    conn.execute("UPDATE tasks SET status='rejected', reject_msg=? WHERE id=?", (reason, tid))
    conn.commit()
    task = conn.execute("SELECT * FROM tasks WHERE id=?", (tid,)).fetchone()
    conn.close()

    await state.clear()
    await msg.answer("✅ Задача отклонена.", reply_markup=ik([("🔙 К задачам", "adm:tasks")]))
    try:
        await bot.send_message(
            task["user_id"],
            f"🔴 Задача <b>#{tid}</b> отклонена.\n💬 Причина: {reason}\n\nИсправь и отправь файлы заново."
        )
    except Exception:
        pass

@router.callback_query(F.data.startswith("adm:task_files:"))
async def cb_task_files(call: CallbackQuery, bot: Bot):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    tid = int(call.data.split(":")[2])

    conn  = get_db()
    files = conn.execute("SELECT * FROM task_files WHERE task_id=?", (tid,)).fetchall()
    conn.close()

    if not files:
        await call.answer("Файлов нет.", show_alert=True)
        return

    await call.answer()
    await call.message.answer(f"📎 Файлы по задаче <b>#{tid}</b>:")
    for f in files:
        await bot.send_document(
            call.from_user.id,
            document=f["file_id"],
            caption=f"📄 {f['file_name']} | Загружен: {f['uploaded_at']}"
        )

# ═══════════════════════════════════════════════════════════════════════════════
#  АДМИН — ПОЛЬЗОВАТЕЛИ
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data == "adm:users")
async def cb_adm_users(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    kb = ik(
        [("📋 Список пользователей", "adm:users_list")],
        [("⏱ Добавить/убрать время", "adm:users_time")],
        [("🔙 Назад", "adm:main")],
    )
    await call.message.edit_text("👥 <b>Пользователи</b>", reply_markup=kb)
    await call.answer()

@router.callback_query(F.data == "adm:users_list")
async def cb_users_list(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return

    conn  = get_db()
    users = conn.execute("SELECT * FROM users ORDER BY joined_at DESC").fetchall()
    conn.close()

    if not users:
        await call.message.edit_text("Нет пользователей.", reply_markup=ik([("🔙 Назад", "adm:users")]))
        await call.answer()
        return

    rows = [[(f"👤 {u['full_name']} (@{u['username']}) #{u['id']}", f"adm:user_profile:{u['id']}")] for u in users]
    rows.append([("🔙 Назад", "adm:users")])
    await call.message.edit_text("👥 <b>Все пользователи:</b>", reply_markup=ik(*rows))
    await call.answer()

@router.callback_query(F.data.startswith("adm:user_profile:"))
async def cb_user_profile(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    uid = int(call.data.split(":")[2])

    conn       = get_db()
    user       = conn.execute("SELECT * FROM users WHERE id=?", (uid,)).fetchone()
    roles      = conn.execute("""
        SELECT r.name, r.stage, p.name AS pname FROM user_roles ur
        JOIN roles r ON r.id=ur.role_id JOIN projects p ON p.id=r.project_id
        WHERE ur.user_id=?
    """, (uid,)).fetchall()
    proj_count = conn.execute("""
        SELECT COUNT(DISTINCT r.project_id) FROM user_roles ur
        JOIN roles r ON r.id=ur.role_id WHERE ur.user_id=?
    """, (uid,)).fetchone()
    total_time = conn.execute(
        "SELECT COALESCE(SUM(hours),0) FROM user_projects_time WHERE user_id=?", (uid,)
    ).fetchone()
    tasks_done = conn.execute(
        "SELECT COUNT(*) FROM tasks WHERE user_id=? AND status='done'", (uid,)
    ).fetchone()
    conn.close()

    if not user:
        await call.message.edit_text("Пользователь не найден.")
        await call.answer()
        return

    roles_txt = "\n".join(f"  • {r['name']} [{r['stage']}] — {r['pname']}" for r in roles) or "  — нет"
    text = (
        f"👤 <b>{user['full_name']}</b>\n"
        f"@{user['username']} | <code>{user['id']}</code>\n"
        f"📅 В боте с: {user['joined_at']}\n\n"
        f"🏷 Роли:\n{roles_txt}\n\n"
        f"📁 Проектов: {proj_count[0]}\n"
        f"⏱ Время в тиме: {total_time[0]:.1f} ч.\n"
        f"✅ Выполнено задач: {tasks_done[0]}"
    )
    await call.message.edit_text(text, reply_markup=ik([("🔙 К списку", "adm:users_list")]))
    await call.answer()

@router.callback_query(F.data == "adm:users_time")
async def cb_users_time_start(call: CallbackQuery, state: FSMContext):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return
    await state.set_state(UserStates.waiting_time_uid)
    await call.message.edit_text("⏱ Введите <b>Telegram ID</b> пользователя для изменения времени:")
    await call.answer()

@router.message(UserStates.waiting_time_uid)
async def users_time_uid(msg: Message, state: FSMContext):
    if not msg.text.strip().lstrip("-").isdigit():
        await msg.answer("⚠️ Введите числовой ID.")
        return
    uid = int(msg.text.strip())

    conn  = get_db()
    user  = conn.execute("SELECT * FROM users WHERE id=?", (uid,)).fetchone()
    total = conn.execute(
        "SELECT COALESCE(SUM(hours),0) FROM user_projects_time WHERE user_id=?", (uid,)
    ).fetchone()
    conn.close()

    if not user:
        await msg.answer("❌ Пользователь не найден.")
        return

    await state.update_data(time_uid=uid)
    await state.set_state(UserStates.waiting_time_delta)
    await msg.answer(
        f"⏱ Пользователь: <b>{user['full_name']}</b>\n"
        f"Текущее время: <b>{total[0]:.1f} ч.</b>\n\n"
        f"Введите количество часов (положительное — добавить, отрицательное — убрать).\n"
        f"Пример: <code>5</code> или <code>-2.5</code>"
    )

@router.message(UserStates.waiting_time_delta)
async def users_time_delta(msg: Message, state: FSMContext):
    try:
        delta = float(msg.text.strip().replace(",", "."))
    except ValueError:
        await msg.answer("⚠️ Введите число. Пример: 5 или -2.5")
        return

    data = await state.get_data()
    uid  = data["time_uid"]

    conn = get_db()
    row  = conn.execute(
        "SELECT id, hours FROM user_projects_time WHERE user_id=? AND project_id IS NULL", (uid,)
    ).fetchone()

    if row:
        new_hours = max(0.0, row["hours"] + delta)
        conn.execute("UPDATE user_projects_time SET hours=? WHERE id=?", (new_hours, row["id"]))
    else:
        new_hours = max(0.0, delta)
        conn.execute(
            "INSERT INTO user_projects_time(user_id, project_id, hours) VALUES(?,NULL,?)",
            (uid, new_hours)
        )
    conn.commit()
    conn.close()

    await state.clear()
    sign = "+" if delta >= 0 else ""
    await msg.answer(
        f"✅ Время обновлено!\n{sign}{delta} ч. → итого: <b>{new_hours:.1f} ч.</b>",
        reply_markup=ik([("🔙 К пользователям", "adm:users")])
    )

# ═══════════════════════════════════════════════════════════════════════════════
#  АДМИН — ФАЙЛЫ
# ═══════════════════════════════════════════════════════════════════════════════
@router.callback_query(F.data == "adm:files")
async def cb_adm_files(call: CallbackQuery):
    if not is_admin(call.from_user.id): await call.answer("⛔"); return

    conn  = get_db()
    tasks = conn.execute("""
        SELECT t.id, u.full_name, COUNT(tf.id) AS fcount
        FROM tasks t
        JOIN task_files tf ON tf.task_id=t.id
        LEFT JOIN users u ON u.id=t.user_id
        WHERE t.status='pending_review'
        GROUP BY t.id
        ORDER BY t.id DESC
    """).fetchall()
    conn.close()

    if not tasks:
        await call.message.edit_text(
            "📭 Нет задач с файлами на проверке.",
            reply_markup=ik([("🔙 Назад", "adm:main")])
        )
        await call.answer()
        return

    rows = [[(f"📎 Задача #{t['id']} — {t['full_name']} ({t['fcount']} файл.)",
              f"adm:task_files:{t['id']}")] for t in tasks]
    rows.append([("🔙 Назад", "adm:main")])
    await call.message.edit_text("📎 <b>Файлы на проверке:</b>", reply_markup=ik(*rows))
    await call.answer()

# ═══════════════════════════════════════════════════════════════════════════════
#  ЗАПУСК
# ═══════════════════════════════════════════════════════════════════════════════
async def main():
    init_db()           # синхронный — без await!

    from aiogram.client.default import DefaultBotProperties
bot = Bot(token=BOT_TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
    dp  = Dispatcher(storage=MemoryStorage())
    dp.include_router(router)

    await bot.set_my_commands([
        BotCommand(command="start", description="▶️ Запустить бота"),
        BotCommand(command="admin", description="🛠 Панель администратора"),
    ])

    log.info("🚀 BALBES TEAM Bot запущен!")
    await dp.start_polling(bot, allowed_updates=dp.resolve_used_update_types())

if __name__ == "__main__":
    asyncio.run(main())
