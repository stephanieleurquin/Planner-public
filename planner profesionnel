import streamlit as st
import sqlite3
import pandas as pd
from datetime import datetime, timedelta
import plotly.express as px
import plotly.graph_objects as go
import os

# Configuration de la page
st.set_page_config(
    page_title="PlannerPro Dashboard",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

DB_NAME = "plannerpro.db"

# CSS personnalisé
st.markdown("""
    <style>
    .main-header {
        font-size: 2.5rem;
        color: #1f77b4;
        text-align: center;
        margin-bottom: 1.5rem;
        font-weight: bold;
    }
    .stat-card {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        padding: 1.5rem;
        border-radius: 15px;
        text-align: center;
        color: white;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        transition: transform 0.3s;
    }
    .stat-card:hover {
        transform: translateY(-5px);
    }
    .stat-number {
        font-size: 2.8rem;
        font-weight: bold;
        margin: 0.5rem 0;
    }
    .stat-label {
        font-size: 1rem;
        opacity: 0.9;
    }
    .task-card {
        background-color: white;
        padding: 1rem;
        border-radius: 10px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        margin: 0.5rem 0;
        border-left: 4px solid #1f77b4;
    }
    .task-done {
        border-left-color: #28a745;
    }
    .task-pending {
        border-left-color: #dc3545;
    }
    .task-high {
        border-left-color: #dc3545;
    }
    .task-medium {
        border-left-color: #ffc107;
    }
    .task-low {
        border-left-color: #28a745;
    }
    .sidebar-content {
        padding: 1rem;
    }
    .filter-section {
        background-color: #f8f9fa;
        padding: 1rem;
        border-radius: 10px;
        margin: 1rem 0;
    }
    .stButton > button {
        width: 100%;
        border-radius: 8px;
        font-weight: bold;
    }
    .priority-high { color: #dc3545; }
    .priority-medium { color: #ffc107; }
    .priority-low { color: #28a745; }
    .footer {
        text-align: center;
        color: #6c757d;
        padding: 1rem;
        margin-top: 2rem;
        border-top: 1px solid #dee2e6;
    }
    </style>
""", unsafe_allow_html=True)


# Fonctions de base de données
def get_db():
    conn = sqlite3.connect(DB_NAME)
    conn.row_factory = sqlite3.Row
    return conn


def init_db():
    conn = get_db()
    conn.execute("""
                 CREATE TABLE IF NOT EXISTS tasks
                 (
                     id
                     INTEGER
                     PRIMARY
                     KEY
                     AUTOINCREMENT,
                     text
                     TEXT
                     NOT
                     NULL,
                     done
                     INTEGER
                     DEFAULT
                     0,
                     category
                     TEXT
                     DEFAULT
                     'day',
                     created_at
                     TEXT
                     DEFAULT
                     CURRENT_TIMESTAMP,
                     priority
                     TEXT
                     DEFAULT
                     'medium',
                     due_date
                     TEXT
                 )
                 """)
    conn.commit()
    conn.close()


def load_data():
    conn = get_db()
    df = pd.read_sql_query("""
                           SELECT id,
                                  text,
                                  done,
                                  category,
                                  created_at,
                                  priority,
                                  due_date,
                                  CASE WHEN done = 1 THEN '✅ Terminé' ELSE '⏳ En cours' END as status
                           FROM tasks
                           ORDER BY id DESC
                           """, conn)
    conn.close()

    if not df.empty:
        df['created_at'] = pd.to_datetime(df['created_at'])
        df['due_date'] = pd.to_datetime(df['due_date'], errors='coerce')

    return df


def get_stats(df):
    total = len(df)
    done = len(df[df['done'] == 1]) if not df.empty else 0
    pending = total - done
    completion_rate = (done / total * 100) if total > 0 else 0

    return {
        'total': total,
        'done': done,
        'pending': pending,
        'completion_rate': completion_rate
    }


# Initialisation de la base de données
init_db()

# Sidebar
with st.sidebar:
    st.markdown("""
        <div style="text-align: center; padding: 1rem 0;">
            <h1 style="font-size: 2rem; margin: 0;">📊</h1>
            <h2 style="margin: 0; color: #1f77b4;">PlannerPro</h2>
            <p style="color: #6c757d; margin: 0;">v2.0 Dashboard</p>
        </div>
    """, unsafe_allow_html=True)

    st.markdown("---")

    # Section Ajout rapide
    st.subheader("➕ Nouvelle Tâche")
    with st.form("quick_add", clear_on_submit=True):
        task_text = st.text_input("✏️ Texte de la tâche", placeholder="Entrez votre tâche...")

        col1, col2 = st.columns(2)
        with col1:
            task_category = st.selectbox("📂 Catégorie", ['day', 'week', 'month'])
        with col2:
            task_priority = st.selectbox("🎯 Priorité", ['low', 'medium', 'high'])

        task_due = st.date_input("📅 Date limite", value=None, help="Optionnel")

        submitted = st.form_submit_button("✅ Ajouter", use_container_width=True)

        if submitted and task_text:
            conn = get_db()
            due_date = task_due.strftime('%Y-%m-%d') if task_due else None
            conn.execute(
                "INSERT INTO tasks (text, category, priority, due_date) VALUES (?,?,?,?)",
                (task_text, task_category, task_priority, due_date)
            )
            conn.commit()
            conn.close()
            st.success(f"✅ Tâche ajoutée avec succès !")
            st.rerun()
        elif submitted and not task_text:
            st.error("⚠️ Veuillez entrer un texte pour la tâche")

    st.markdown("---")

    # Section Filtres
    st.subheader("🔍 Filtres")

    with st.expander("Catégories", expanded=True):
        category_filter = st.multiselect(
            "Sélectionner",
            options=['day', 'week', 'month'],
            default=['day', 'week', 'month']
        )

    with st.expander("Statut", expanded=True):
        status_filter = st.multiselect(
            "Sélectionner",
            options=['✅ Terminé', '⏳ En cours'],
            default=['✅ Terminé', '⏳ En cours']
        )

    with st.expander("Priorité", expanded=True):
        priority_filter = st.multiselect(
            "Sélectionner",
            options=['low', 'medium', 'high'],
            default=['low', 'medium', 'high']
        )

    st.markdown("---")

    # Actions rapides
    st.subheader("⚡ Actions")

    if st.button("✅ Marquer tout comme fait", use_container_width=True):
        conn = get_db()
        conn.execute("UPDATE tasks SET done = 1")
        conn.commit()
        conn.close()
        st.success("✅ Toutes les tâches marquées comme faites !")
        st.rerun()

    if st.button("🗑️ Supprimer terminées", use_container_width=True):
        conn = get_db()
        conn.execute("DELETE FROM tasks WHERE done = 1")
        conn.commit()
        conn.close()
        st.success("🗑️ Tâches terminées supprimées !")
        st.rerun()

    if st.button("🔄 Réinitialiser DB", use_container_width=True):
        conn = get_db()
        conn.execute("DROP TABLE IF EXISTS tasks")
        conn.commit()
        conn.close()
        init_db()
        st.warning("🔄 Base de données réinitialisée")
        st.rerun()

# Chargement des données
df = load_data()

# Application des filtres
if not df.empty:
    filtered_df = df[
        (df['category'].isin(category_filter)) &
        (df['status'].isin(status_filter)) &
        (df['priority'].isin(priority_filter))
        ]
else:
    filtered_df = df

# En-tête principal
st.markdown('<h1 class="main-header">📊 Tableau de Bord PlannerPro</h1>', unsafe_allow_html=True)

# Statistiques
stats = get_stats(filtered_df)

# Colonnes de statistiques avec couleurs différentes
col1, col2, col3, col4 = st.columns(4)

with col1:
    st.markdown(f"""
        <div class="stat-card" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
            <div class="stat-label">📋 Total Tâches</div>
            <div class="stat-number">{stats['total']}</div>
        </div>
    """, unsafe_allow_html=True)

with col2:
    st.markdown(f"""
        <div class="stat-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
            <div class="stat-label">✅ Terminées</div>
            <div class="stat-number">{stats['done']}</div>
        </div>
    """, unsafe_allow_html=True)

with col3:
    st.markdown(f"""
        <div class="stat-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
            <div class="stat-label">⏳ En Cours</div>
            <div class="stat-number">{stats['pending']}</div>
        </div>
    """, unsafe_allow_html=True)

with col4:
    color = "#28a745" if stats['completion_rate'] > 70 else "#ffc107" if stats['completion_rate'] > 40 else "#dc3545"
    st.markdown(f"""
        <div class="stat-card" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
            <div class="stat-label">📈 Taux Complétion</div>
            <div class="stat-number">{stats['completion_rate']:.1f}%</div>
        </div>
    """, unsafe_allow_html=True)

st.markdown("---")

# Visualisations
if not filtered_df.empty:
    col1, col2 = st.columns(2)

    with col1:
        st.subheader("📊 Répartition par Catégorie")
        cat_counts = filtered_df['category'].value_counts()
        fig = px.pie(
            values=cat_counts.values,
            names=cat_counts.index,
            color=cat_counts.index,
            color_discrete_map={'day': '#1f77b4', 'week': '#ff7f0e', 'month': '#2ca02c'},
            hole=0.3
        )
        fig.update_traces(textposition='inside', textinfo='percent+label')
        fig.update_layout(height=350)
        st.plotly_chart(fig, use_container_width=True)

    with col2:
        st.subheader("📊 Statut des Tâches")
        status_counts = filtered_df['status'].value_counts()
        fig = px.bar(
            x=status_counts.index,
            y=status_counts.values,
            color=status_counts.index,
            color_discrete_map={'✅ Terminé': '#28a745', '⏳ En cours': '#dc3545'},
            text=status_counts.values
        )
        fig.update_traces(textposition='outside')
        fig.update_layout(
            showlegend=False,
            xaxis_title="",
            yaxis_title="Nombre de tâches",
            height=350
        )
        st.plotly_chart(fig, use_container_width=True)

    # Graphique de priorité
    st.subheader("🎯 Distribution par Priorité")
    priority_counts = filtered_df['priority'].value_counts()

    col1, col2, col3 = st.columns(3)
    for idx, (priority, color, emoji) in enumerate([
        ('high', '#dc3545', '🔴'),
        ('medium', '#ffc107', '🟡'),
        ('low', '#28a745', '🟢')
    ]):
        count = priority_counts.get(priority, 0)
        with [col1, col2, col3][idx]:
            st.markdown(f"""
                <div style="background-color: #f8f9fa; padding: 1rem; border-radius: 10px; text-align: center; border: 2px solid {color};">
                    <div style="font-size: 2rem;">{emoji}</div>
                    <div style="font-size: 1.2rem; font-weight: bold; color: {color};">{priority.capitalize()}</div>
                    <div style="font-size: 2rem; font-weight: bold;">{count}</div>
                </div>
            """, unsafe_allow_html=True)

st.markdown("---")

# Liste des tâches
st.subheader("📋 Liste des Tâches")

# Message si aucune tâche
if filtered_df.empty:
    st.info("📭 Aucune tâche trouvée avec les filtres actuels")
else:
    # Affichage des tâches
    for idx, row in filtered_df.iterrows():
        # Déterminer la classe CSS en fonction du statut et priorité
        task_class = "task-card"
        if row['done']:
            task_class += " task-done"
        else:
            task_class += " task-pending"

        if row['priority'] == 'high':
            task_class += " task-high"
        elif row['priority'] == 'medium':
            task_class += " task-medium"
        else:
            task_class += " task-low"

        with st.container():
            col1, col2, col3, col4, col5, col6, col7 = st.columns([0.5, 3, 1, 1, 1, 0.5, 0.5])

            with col1:
                done = st.checkbox(
                    "✅",
                    value=bool(row['done']),
                    key=f"done_{row['id']}",
                    label_visibility="collapsed"
                )
                if done != bool(row['done']):
                    conn = get_db()
                    conn.execute("UPDATE tasks SET done = ? WHERE id = ?", (int(done), row['id']))
                    conn.commit()
                    conn.close()
                    st.rerun()

            with col2:
                if row['done']:
                    st.markdown(f"<div style='text-decoration: line-through; color: #6c757d;'>{row['text']}</div>",
                                unsafe_allow_html=True)
                else:
                    st.markdown(f"<div style='font-weight: 500;'>{row['text']}</div>", unsafe_allow_html=True)

            with col3:
                emoji = {"day": "📅", "week": "📆", "month": "📆"}.get(row['category'], "📌")
                st.write(f"{emoji} {row['category']}")

            with col4:
                priority_emoji = {"high": "🔴", "medium": "🟡", "low": "🟢"}.get(row['priority'], "⚪")
                st.write(f"{priority_emoji} {row['priority']}")

            with col5:
                if row['due_date'] and pd.notna(row['due_date']):
                    days_left = (row['due_date'] - datetime.now()).days
                    if days_left < 0:
                        st.error(f"📅 {days_left}j")
                    elif days_left == 0:
                        st.warning("📅 Aujourd'hui")
                    else:
                        st.info(f"📅 {days_left}j")
                else:
                    st.write("—")

            with col6:
                if st.button("✏️", key=f"edit_{row['id']}", help="Modifier"):
                    st.session_state.editing = row['id']
                    st.session_state.edit_text = row['text']

            with col7:
                if st.button("🗑️", key=f"del_{row['id']}", help="Supprimer"):
                    conn = get_db()
                    conn.execute("DELETE FROM tasks WHERE id = ?", (row['id'],))
                    conn.commit()
                    conn.close()
                    st.rerun()

# Pied de page
st.markdown("""
    <div class="footer">
        <p>🔄 PlannerPro Dashboard v2.0 | Dernière mise à jour : {}</p>
        <p style="font-size: 0.8rem; opacity: 0.7;">Développé avec ❤️ en Python & Streamlit</p>
    </div>
""".format(datetime.now().strftime('%Y-%m-%d %H:%M:%S')), unsafe_allow_html=True)
