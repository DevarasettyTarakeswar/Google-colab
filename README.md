# Google-colab
A futuristic, AI-powered career recommendation web app built with Python &amp; Gradio. Designed to run directly in Google Colab — no local setup required.

!pip install -q gradio pandas matplotlib


import gradio as gr
import pandas as pd
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import matplotlib.patches as patches
import numpy as np
import os, csv, io, traceback
from datetime import datetime


SKILL_LIST = [
    "Python", "Machine Learning", "Deep Learning", "Data Analysis",
    "SQL", "Excel", "Power BI", "Tableau", "NLP", "Computer Vision",
    "Cloud (AWS/GCP/Azure)", "Docker", "Kubernetes", "Java", "JavaScript",
    "React", "Node.js", "C++", "Cybersecurity", "Networking",
    "Project Management", "Communication", "Leadership", "Agile/Scrum",
    "Statistics", "R", "TensorFlow", "PyTorch", "Prompt Engineering",
    "Blockchain", "IoT", "AR/VR Development", "Quantum Computing",
    # New skills added by user request
    "Accounting", "Financial Reporting", "Auditing", "Tax", "Financial Analysis",
    "Financial Modeling", "Risk Management", "Research", "Econometrics", "Business Strategy",
    "Strategic Planning", "Digital Marketing", "Market Research", "Analytics", "Sales",
    "Negotiation", "Social Media Marketing", "SEO", "SEM", "Content Marketing",
    "Patient Care", "Medical Terminology", "Health Informatics", "Clinical Research",
    "Wellness Coaching", "Nutrition", "Psychology", "Graphic Design", "UI/UX Design",
    "Content Creation", "Video Editing", "Illustration", "Copywriting", "Creative Direction",
    "Digital Art", "Curriculum Development", "Pedagogy", "Research Methodology",
    "Academic Writing", "Statistical Analysis", "Grant Writing", "Mentoring",
    "Logistics", "Supply Chain Management", "Process Optimization", "Office Management",
    "Customer Service", "Data Entry", "Scheduling", "Problem Solving"
]

PRESENT_JOBS = [
    {"role":"Data Analyst",          "skills":["Python","SQL","Excel","Statistics","Power BI","Tableau"]},
    {"role":"Data Scientist",        "skills":["Python","Machine Learning","Statistics","SQL","TensorFlow","R"]},
    {"role":"ML Engineer",           "skills":["Python","Machine Learning","Deep Learning","TensorFlow","PyTorch","Docker"]},
    {"role":"Software Developer",    "skills":["Python","Java","JavaScript","React","Node.js","Docker"]},
    {"role":"BI Developer",          "skills":["Power BI","Tableau","SQL","Excel","Data Analysis"]},
    {"role":"DevOps Engineer",       "skills":["Docker","Kubernetes","Cloud (AWS/GCP/Azure)","Networking"]},
    {"role":"Cybersecurity Analyst", "skills":["Cybersecurity","Networking","Python","SQL"]},
    {"role":"NLP Engineer",          "skills":["NLP","Python","Deep Learning","TensorFlow","PyTorch"]},
    {"role":"Computer Vision Eng",   "skills":["Computer Vision","Python","Deep Learning","TensorFlow","C++"]},
    {"role":"Project Manager",       "skills":["Project Management","Communication","Leadership","Agile/Scrum","Excel"]},
    {"role":"Cloud Architect",       "skills":["Cloud (AWS/GCP/Azure)","Docker","Kubernetes","Networking","Python"]},
    {"role":"Frontend Developer",    "skills":["JavaScript","React","Node.js","Communication"]},
    {"role":"Database Admin",        "skills":["SQL","Python","Excel","Cloud (AWS/GCP/Azure)"]},
    {"role":"Research Scientist",    "skills":["Python","R","Statistics","Machine Learning","Deep Learning","PyTorch"]},
    # New present jobs added by user request
    {"role":"Investment Banker",       "skills":["Financial Analysis","Communication","Leadership","Project Management","Excel","Statistics","Data Analysis","SQL"]},
    {"role":"Financial Analyst",       "skills":["Financial Analysis","Excel","Statistics","Data Analysis","Communication","SQL","Power BI","Tableau"]},
    {"role":"Chartered Accountant",    "skills":["Accounting","Financial Reporting","Auditing","Tax","Excel","Communication","Leadership"]},
    {"role":"Actuary",                 "skills":["Statistics","Data Analysis","R","Python","Excel","Communication","Financial Modeling","Risk Management"]},
    {"role":"Economist",               "skills":["Statistics","Data Analysis","R","Python","Communication","Research","Econometrics"]},
    {"role":"Management Consultant",   "skills":["Project Management","Communication","Leadership","Data Analysis","Excel","Business Strategy"]},
    {"role":"Marketing Manager",       "skills":["Communication","Leadership","Project Management","Digital Marketing","Market Research","Analytics"]},
    {"role":"Business Development Executive", "skills":["Communication","Leadership","Sales","Negotiation","Strategic Planning","Project Management"]},
    {"role":"Digital Marketing Strategist", "skills":["Digital Marketing","Social Media Marketing","SEO","SEM","Analytics","Content Marketing","Communication"]},
    {"role":"Sales Director",          "skills":["Sales","Leadership","Communication","Negotiation","Strategic Planning","Data Analysis"]},
    {"role":"Financial Advisor",       "skills":["Financial Planning","Investment Analysis","Communication","Negotiation"]},
    {"role":"Business Analyst",        "skills":["Market Analysis","Business Strategy","Data Analysis","Communication","Project Management"]},
    {"role":"Compliance Officer",      "skills":["Compliance","Risk Management","Communication","Project Management"]},
    {"role":"Healthcare Administrator", "skills":["Project Management","Communication","Business Strategy","Health Informatics","Leadership"]},
    {"role":"Clinical Research Coordinator", "skills":["Clinical Research","Data Analysis","Communication","Project Management","Medical Terminology"]},
    {"role":"Wellness Coach",          "skills":["Wellness Coaching","Nutrition","Psychology","Communication"]},
    {"role":"Graphic Designer",        "skills":["Graphic Design","Content Creation","Communication","Creative Direction"]},
    {"role":"UI/UX Designer",          "skills":["UI/UX Design","Communication","Research","Project Management"]},
    {"role":"Content Creator",         "skills":["Content Creation","Copywriting","Video Editing","Social Media Marketing"]},
    {"role":"Educator",                "skills":["Curriculum Development","Pedagogy","Communication","Mentoring"]},
    {"role":"Academic Researcher",     "skills":["Research Methodology","Statistical Analysis","Academic Writing","Grant Writing"]},
    {"role":"Instructional Designer",  "skills":["Curriculum Development","Pedagogy","Content Creation","Project Management"]},
    {"role":"Operations Manager",      "skills":["Logistics","Supply Chain Management","Process Optimization","Project Management","Leadership"]},
    {"role":"Administrative Assistant","skills":["Office Management","Scheduling","Communication","Data Entry"]},
    {"role":"Customer Success Manager","skills":["Customer Service","Communication","Problem Solving","Negotiation","Leadership"]}
]

FUTURE_JOBS = [
    {"role":"AI/LLM Engineer",        "skills":["Prompt Engineering","Python","Deep Learning","NLP","TensorFlow","PyTorch"],
     "gap":["Prompt Engineering","LLM Fine-tuning","RLHF"]},
    {"role":"Generative AI Specialist","skills":["Prompt Engineering","Python","Deep Learning","NLP","Computer Vision"],
     "gap":["Diffusion Models","Multimodal AI","Prompt Engineering"]},
    {"role":"Quantum ML Researcher",  "skills":["Quantum Computing","Python","Machine Learning","Statistics","R"],
     "gap":["Quantum Computing","Qiskit","Quantum Algorithms"]},
    {"role":"AR/VR Engineer",         "skills":["AR/VR Development","Python","C++","Computer Vision","JavaScript"],
     "gap":["AR/VR Development","Unity/Unreal","Spatial Computing"]},
    {"role":"Blockchain Developer",   "skills":["Blockchain","Python","JavaScript","Cybersecurity"],
     "gap":["Blockchain","Smart Contracts","Web3.js"]},
    {"role":"IoT Architect",          "skills":["IoT","Python","Networking","Cloud (AWS/GCP/Azure)","C++"],
     "gap":["IoT","Edge Computing","Embedded Systems"]},
    {"role":"AI Ethics Consultant",   "skills":["Machine Learning","Communication","Leadership","Statistics","Python"],
     "gap":["AI Governance","Fairness Auditing","Policy Writing"]},
    {"role":"MLOps Engineer",         "skills":["Docker","Kubernetes","Python","Machine Learning","Cloud (AWS/GCP/Azure)"],
     "gap":["MLflow","Feature Stores","Model Monitoring"]},
    {"role":"Edge AI Engineer",       "skills":["Deep Learning","Python","IoT","C++","TensorFlow"],
     "gap":["TensorFlow Lite","ONNX","Edge Inference"]},
    {"role":"Autonomous Systems Eng", "skills":["Computer Vision","Deep Learning","Python","C++","IoT"],
     "gap":["ROS","Control Systems","Sensor Fusion"]},
]

ROADMAP = {
    "Phase 1  Foundation":   ["Python","SQL","Statistics","Excel","Communication"],
    "Phase 2  Core AI/Data": ["Machine Learning","Data Analysis","Power BI","Tableau","TensorFlow"],
    "Phase 3  Advanced AI":  ["Deep Learning","NLP","Computer Vision","PyTorch","Cloud (AWS/GCP/Azure)"],
    "Phase 4  Future Tech":  ["Prompt Engineering","Quantum Computing","AR/VR Development","Blockchain","IoT"],
}

def get_skills(dd, manual):
    skills = set(dd or [])
    if manual:
        for s in manual.split(","):
            s = s.strip()
            if s:
                skills.add(s)
    return list(skills)

def match_present(user_skills):
    uset = {s.lower() for s in user_skills}
    out = []
    for j in PRESENT_JOBS:
        req = {s.lower() for s in j["skills"]}
        ov  = uset & req
        sc  = len(ov) / len(req)
        if sc >= 0.25:
            out.append({"Role": j["role"],
                        "Matched Skills": ", ".join(sorted(ov)),
                        "Match %": f"{sc*100:.0f}%",
                        "_score": sc})
    return sorted(out, key=lambda x: -x["_score"])

def match_future(user_skills):
    uset = {s.lower() for s in user_skills}
    out = []
    for j in FUTURE_JOBS:
        req = {s.lower() for s in j["skills"]}
        ov  = uset & req
        sc  = len(ov) / len(req) if req else 0
        if sc >= 0.20:
            gaps = [g for g in j["gap"] if g.lower() not in uset]
            out.append({"Role": j["role"],
                        "Matched Skills": ", ".join(sorted(ov)),
                        "Match %": f"{sc*100:.0f}%",
                        "Skills to Learn": ", ".join(gaps) if gaps else "All covered!",
                        "_score": sc})
    return sorted(out, key=lambda x: -x["_score"])

def make_report(user_skills, present, future):
    ts  = datetime.now().strftime("%Y-%m-%d %H:%M")
    sep = "=" * 55
    L = [sep, "  CAREER SCENARIO COMPARISON REPORT",
         f"  Generated : {ts}", sep, "",
         f"YOUR SKILLS ({len(user_skills)}):",
         "  " + ", ".join(user_skills) if user_skills else "  None", "",
         "-" * 55, f"PRESENT JOBS MATCHED  ({len(present)})", "-" * 55]
    for j in present[:8]:
        L += [f"  + {j['Role']:<30} {j['Match %']}",
              f"    Skills : {j['Matched Skills']}"]
    if not present:
        L.append("  None matched. Add more skills!")
    L += ["", "-" * 55, f"FUTURE JOBS MATCHED  ({len(future)})", "-" * 55]
    for j in future[:8]:
        L += [f"  * {j['Role']:<30} {j['Match %']}",
              f"    Matched : {j['Matched Skills']}",
              f"    To Learn: {j['Skills to Learn']}"]
    if not future:
        L.append("  None matched. Consider upskilling!")
    gaps = set()
    for j in future:
        for s in j["Skills to Learn"].split(", "):
            if s and s != "All covered!":
                gaps.add(s)
    L += ["", "-" * 55, f"SKILL GAPS  ({len(gaps)} missing)", "-" * 55,
          "  " + (", ".join(sorted(gaps)) if gaps else "None — well equipped!")]
    uset = {s.lower() for s in user_skills}
    L += ["", "-" * 55, "ROADMAP", "-" * 55]
    for phase, skills in ROADMAP.items():
        have = [s for s in skills if s.lower() in uset]
        need = [s for s in skills if s.lower() not in uset]
        pct  = int(len(have)/len(skills)*100)
        bar  = "#"*(pct//10) + "."*(10-pct//10)
        L += [f"\n  {phase}  [{bar}] {pct}%",
              f"    Have : {', '.join(have) if have else 'None'}",
              f"    Need : {', '.join(need) if need else 'Complete!'}"]
    L += ["", sep, "END OF REPORT", sep]
    return "\n".join(L)

def make_roadmap_text(user_skills):
    uset = {s.lower() for s in user_skills}
    L = ["=" * 55, "  PERSONALISED SKILL-BUILDING ROADMAP", "=" * 55, ""]
    for phase, skills in ROADMAP.items():
        have = [s for s in skills if s.lower() in uset]
        need = [s for s in skills if s.lower() not in uset]
        pct  = int(len(have)/len(skills)*100)
        bar  = "#"*(pct//10) + "."*(10-pct//10)
        L += [f"  {phase}",
              f"  Progress  [{bar}] {pct}%",
              f"  Have    : {', '.join(have) if have else 'None'}",
              f"  Need    : {', '.join(need) if need else 'Complete!'}",
              "  " + "-"*51, ""]
    for phase, skills in ROADMAP.items():
        need = [s for s in skills if s.lower() not in uset]
        if need:
            L += ["  NEXT STEP", "  " + "-"*51,
                  f"  Start with : {need[0]}  ({phase.strip()})"]
            if len(need) > 1:
                L.append(f"  Then learn : {', '.join(need[1:3])}")
            break
    return "\n".join(L)

# ── Chart — always returns a valid png path 
def make_chart(present, future):
    path = "/tmp/career_chart.png"
    try:
        fig, axes = plt.subplots(1, 2, figsize=(13, 5.5))
        fig.patch.set_facecolor("#050d1a")

        def hbar(ax, data, title, col):
            ax.set_facecolor("#050d1a")
            if not data:
                ax.text(0.5, 0.5, "No matches yet",
                        color="#4af6c3", ha="center", va="center",
                        fontsize=11, transform=ax.transAxes,
                        fontfamily="monospace")
            else:
                items  = data[:7]
                roles  = [d["Role"]    for d in items]
                scores = [d["_score"]  for d in items]
                y = np.arange(len(roles))
                ax.barh(y, scores, color=col, alpha=0.80, height=0.5, zorder=3)
                ax.barh(y, scores, color=col, alpha=0.15, height=0.7, zorder=2)
                for yi, sc in zip(y, scores):
                    ax.text(sc+0.012, yi, f"{sc*100:.0f}%",
                            va="center", color="#ddf4ff",
                            fontsize=8.5, fontweight="bold")
                ax.set_yticks(y)
                ax.set_yticklabels(roles, color="#a8d8ff", fontsize=8.5)
                ax.set_xlim(0, 1.22)
                ax.set_xticks([0,.25,.5,.75,1])
                ax.set_xticklabels(["0%","25%","50%","75%","100%"],
                                   color="#4af6c3", fontsize=7.5)
            for sp in ax.spines.values():
                sp.set_edgecolor("#1a3a5c")
            ax.tick_params(length=0)
            ax.grid(axis="x", color="#0d2a40", linewidth=0.6,
                    linestyle="--", zorder=0)
            ax.set_title(title, color="#4af6c3", fontsize=12,
                         fontweight="bold", pad=10, fontfamily="monospace")

        hbar(axes[0], present, "Present Job Matches", "#00c6ff")
        hbar(axes[1], future,  "Future Job Matches",  "#a855f7")
        fig.suptitle("CAREER MATCH VISUALIZATION", color="#4af6c3",
                     fontsize=14, fontweight="bold", y=1.02,
                     fontfamily="monospace")
        plt.tight_layout()
        plt.savefig(path, dpi=130, bbox_inches="tight",
                    facecolor=fig.get_facecolor())
        plt.close(fig)
    except Exception as e:
        # Fallback: plain dark image with error text
        fig, ax = plt.subplots(figsize=(8, 4))
        fig.patch.set_facecolor("#050d1a")
        ax.set_facecolor("#050d1a")
        ax.text(0.5, 0.5, f"Chart error:\n{e}", color="#ff6b6b",
                ha="center", va="center", transform=ax.transAxes)
        plt.savefig(path, dpi=100, facecolor=fig.get_facecolor())
        plt.close(fig)
    return path

# ── CSV — always returns a valid file path 
def make_csv(future):
    path = "/tmp/future_careers.csv"
    try:
        rows = [{"Role": j["Role"],
                 "Match %": j["Match %"],
                 "Matched Skills": j["Matched Skills"],
                 "Skills to Learn": j["Skills to Learn"]}
                for j in future]
        if not rows:
            rows = [{"Role":"No matches","Match %":"","Matched Skills":"","Skills to Learn":"Add more skills"}]
        with open(path, "w", newline="", encoding="utf-8") as f:
            w = csv.DictWriter(f, fieldnames=["Role","Match %","Matched Skills","Skills to Learn"])
            w.writeheader()
            w.writerows(rows)
    except Exception:
        with open(path, "w") as f:
            f.write("Role,Match %,Matched Skills,Skills to Learn\n")
            f.write("Error generating CSV,,, \n")
    return path

# ── Report TXT — always returns a valid file path
def make_txt(report_text):
    path = "/tmp/career_report.txt"
    try:
        with open(path, "w", encoding="utf-8") as f:
            f.write(report_text)
    except Exception:
        with open(path, "w") as f:
            f.write("Report generation error.")
    return path

EMPTY_P = pd.DataFrame(columns=["Role","Matched Skills","Match %"])
EMPTY_F = pd.DataFrame(columns=["Role","Matched Skills","Match %","Skills to Learn"])

def find_careers(dd_skills, manual_skills):
    try:
        user_skills = get_skills(dd_skills, manual_skills)
        if not user_skills:
            return (EMPTY_P.copy(), EMPTY_F.copy(),
                    "Please enter at least one skill.",
                    make_chart([], []),
                    make_csv([]),
                    make_txt("No skills entered."),
                    "Enter skills above and click FIND CAREERS")

        present = match_present(user_skills)
        future  = match_future(user_skills)

        # Build DataFrames — drop internal _score column
        df_p = (pd.DataFrame([{k:v for k,v in r.items() if k!="_score"}
                               for r in present])
                if present else EMPTY_P.copy())
        df_f = (pd.DataFrame([{k:v for k,v in r.items() if k!="_score"}
                               for r in future])
                if future else EMPTY_F.copy())

        report = make_report(user_skills, present, future)

        gaps = set()
        for j in future:
            for s in j["Skills to Learn"].split(", "):
                if s and s != "All covered!":
                    gaps.add(s)

        kpi = (f"Present Roles: {len(present)}   |   "
               f"Future Roles: {len(future)}   |   "
               f"Skill Gaps: {len(gaps)}")

        chart_path = make_chart(present, future)
        csv_path   = make_csv(future)
        txt_path   = make_txt(report)

        return df_p, df_f, report, chart_path, csv_path, txt_path, kpi

    except Exception as e:
        err = traceback.format_exc()
        return (EMPTY_P.copy(), EMPTY_F.copy(),
                f"ERROR:\n{err}",
                make_chart([], []),
                make_csv([]),
                make_txt(f"ERROR:\n{err}"),
                f"Error: {str(e)}")


def view_roadmap(dd_skills, manual_skills):
    try:
        user_skills = get_skills(dd_skills, manual_skills)
        if not user_skills:
            return "Please enter at least one skill first."
        return make_roadmap_text(user_skills)
    except Exception as e:
        return f"Roadmap error: {e}"

#  CSS
CSS = """
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Share+Tech+Mono&family=Rajdhani:wght@400;500;700&display=swap');

*, body, .gradio-container {
    box-sizing: border-box;
    background-color: #020812;
    font-family: 'Rajdhani', sans-serif;
    color: #c8e6ff;
}

/* background glow */
.gradio-container {
    background:
        radial-gradient(ellipse at 15% 40%, rgba(0,100,255,.06) 0%, transparent 55%),
        radial-gradient(ellipse at 85% 15%, rgba(100,0,255,.06) 0%, transparent 55%),
        #020812 !important;
    min-height: 100vh;
}

/* scan line */
#scanline {
    position:fixed; top:0; left:0; right:0; height:3px; z-index:9999;
    pointer-events:none;
    background:linear-gradient(90deg,transparent,rgba(74,246,195,.55),transparent);
    animation: scan 8s linear infinite;
}
@keyframes scan {
    0%  { top:-4px; opacity:0; }
    5%  { opacity:1; }
    95%  { opacity:1; }
    100%{ top:100vh; opacity:0; }
}

/* header */
#hdr { text-align:center; padding:1.5rem 0 .5rem; }
#htitle {
    font-family:'Orbitron',monospace;
    font-size:clamp(1.6rem,4vw,2.4rem);
    font-weight:900; letter-spacing:.18em;
    background:linear-gradient(90deg,#00c6ff,#4af6c3,#a855f7,#00c6ff);
    background-size:300% 100%;
    -webkit-background-clip:text; -webkit-text-fill-color:transparent;
    animation: shimmer 4s linear infinite;
}
@keyframes shimmer{ to{ background-position:300% 50%; } }
#hsub {
    font-family:'Share Tech Mono',monospace;
    color:#4af6c3; font-size:.78rem; letter-spacing:.28em; opacity:.65;
    margin-top:.35rem;
}

/* panel cards */
.panel {
    background: linear-gradient(135deg,rgba(0,22,55,.93),rgba(4,12,26,.97)) !important;
    border: 1px solid rgba(74,246,195,.2) !important;
    border-radius: 14px !important;
    box-shadow: 0 0 25px rgba(0,198,255,.06) !important;
    padding: 1.2rem !important;
    position: relative; overflow: hidden;
    margin-bottom: .5rem;
}
.panel::before {
    content:''; position:absolute; top:0; left:0; right:0; height:2px;
    background:linear-gradient(90deg,transparent,#4af6c3,#00c6ff,#a855f7,transparent);
}

.slabel {
    font-family:'Orbitron',monospace !important;
    color:#4af6c3 !important; font-size:.68rem !important;
    letter-spacing:.3em !important; margin-bottom:.6rem !important;
}

/* KPI box */
#kpi textarea, #kpi input {
    font-family:'Orbitron',monospace !important;
    font-size:.76rem !important; color:#4af6c3 !important;
    text-align:center; background:rgba(0,198,255,.05) !important;
    border:1px solid rgba(74,246,195,.28) !important;
    border-radius:10px !important; cursor:default !important;
}

/* buttons */
button.lg {
    font-family:'Orbitron',monospace !important;
    font-weight:700 !important; letter-spacing:.1em !important;
    font-size:.82rem !important; border-radius:8px !important;
    transition:all .3s !important;
}
button.lg.primary {
    background:linear-gradient(135deg,#00c6ff 0%,#4af6c3 50%,#a855f7 100%) !important;
    color:#020812 !important; border:none !important;
    box-shadow:0 0 18px rgba(74,246,195,.3) !important;
}
button.lg.primary:hover {
    box-shadow:0 0 30px rgba(74,246,195,.55) !important;
    transform:translateY(-2px) !important;
}
button.lg.secondary {
    background:rgba(0,198,255,.08) !important;
    color:#4af6c3 !important;
    border:1px solid rgba(74,246,195,.35) !important;
}
button.lg.secondary:hover {
    background:rgba(74,246,195,.14) !important;
    box-shadow:0 0 16px rgba(74,246,195,.2) !important;
}

/* inputs / dropdown */
input, textarea {
    background:rgba(0,20,48,.88) !important;
    border:1px solid rgba(0,198,255,.28) !important;
    border-radius:8px !important; color:#c8e6ff !important;
    font-family:'Share Tech Mono',monospace !important;
}
input:focus, textarea:focus {
    border-color:#4af6c3 !important;
    box-shadow:0 0 10px rgba(74,246,195,.18) !important;
    outline:none !important;
}

/* table */
table {
    background:rgba(0,14,34,.92) !important;
    font-family:'Share Tech Mono',monospace !important;
    font-size:.79rem !important; width:100%;
    border-collapse:collapse;
}
th {
    background:rgba(74,246,195,.09) !important;
    color:#4af6c3 !important; font-size:.68rem !important;
    letter-spacing:.1em !important; padding:9px 12px !important;
    font-family:'Orbitron',monospace !important;
}
th { border-bottom:1px solid rgba(74,246,195,.25) !important; }
td { color:#a8d8ff !important; padding:7px 12px !important;
     border-bottom:1px solid rgba(0,80,160,.18) !important; }
tr:hover td { background:rgba(0,198,255,.04) !important; }

/* report / roadmap textbox */
.mono textarea {
    font-family:'Share Tech Mono',monospace !important;
    font-size:.75rem !important; color:#4af6c3 !important;
    background:rgba(0,8,22,.97) !important;
    border:1px solid rgba(74,246,195,.16) !important;
    line-height:1.65 !important;
}

/* tabs */
.tab-nav button {
    font-family:'Orbitron',monospace !important;
    font-size:.65rem !important; letter-spacing:.14em !important;
    color:#557799 !important; background:transparent !important;
    border:none !important; border-bottom:2px solid transparent !important;
    padding:.55rem 1rem !important; transition:all .25s !important;
}
.tab-nav button.selected {
    color:#4af6c3 !important; border-bottom:2px solid #4af6c3 !important;
}

/* image */
img {
    border-radius:12px !important;
    border:1px solid rgba(74,246,195,.2) !important;
    box-shadow:0 0 25px rgba(0,198,255,.09) !important;
}

/* scrollbar */
::-webkit-scrollbar{ width:5px; }
::-webkit-scrollbar-track{ background:#020812; }
::-webkit-scrollbar-thumb{ background:#1a4a7a; border-radius:10px; }
::-webkit-scrollbar-thumb:hover{ background:#4af6c3; }

/* footer */
#footer {
    text-align:center; padding:1rem 0 .5rem;
    font-family:'Share Tech Mono',monospace;
    font-size:.68rem; color:rgba(74,246,195,.32);
    letter-spacing:.2em;
}
"""

#  BUILD GRADIO UI

with gr.Blocks(css=CSS, title="Career OS") as demo:

    # scan line + header
    gr.HTML('<div id="scanline"></div>')
    gr.HTML('''
      <div id="hdr">
        <div id="htitle">◈ CAREER PREDICTION ◈</div>
        <div id="hsub">[ NEURAL CAREER INTELLIGENCE SYSTEM · v3.0 ]</div>
      </div>
    ''')

    # ── Input panel ────
    with gr.Group(elem_classes="panel"):
        gr.HTML('<div class="slabel">⬡ Skill Matrix Input</div>')
        with gr.Row():
            dd = gr.Dropdown(
                choices=SKILL_LIST, multiselect=True,
                label="Select Skills",
                info="Pick one or more from the list",
                scale=2)
            mi = gr.Textbox(
                label="Or type skills manually",
                placeholder="e.g.  Java, SQL, Leadership",
                info="Comma-separated",
                scale=1)
        with gr.Row():
            btn_find = gr.Button("⟫  FIND CAREERS  ⟫", variant="primary",  scale=2)
            btn_road = gr.Button("⟫  VIEW ROADMAP  ⟫", variant="secondary", scale=1)

    # ── KPI strip (single textbox — no error) 
    kpi = gr.Textbox(
        value="◈  Enter skills above and click FIND CAREERS  ◈",
        label="", interactive=False,
        elem_id="kpi", lines=1, max_lines=1)

    # ── Tabs ───────
    with gr.Tabs():

        with gr.Tab("◈ Present Jobs"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Current Role Matches</div>')
                out_present = gr.DataFrame(
                    headers=["Role","Matched Skills","Match %"],
                    interactive=False, wrap=True)

        with gr.Tab("◈ Future Jobs"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Emerging Role Projections</div>')
                out_future = gr.DataFrame(
                    headers=["Role","Matched Skills","Match %","Skills to Learn"],
                    interactive=False, wrap=True)

        with gr.Tab("◈ Visual Charts"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Match Visualization</div>')
                out_chart = gr.Image(type="filepath", show_label=False)

        with gr.Tab("◈ Scenario Report"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Full Comparison Report</div>')
                out_report = gr.Textbox(
                    label="", lines=28, interactive=False,
                    show_copy_button=True, elem_classes="mono")

        with gr.Tab("◈ Roadmap"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Skill-Building Roadmap</div>')
                out_roadmap = gr.Textbox(
                    label="", lines=26, interactive=False,
                    show_copy_button=True, elem_classes="mono")

        with gr.Tab("◈ Downloads"):
            with gr.Group(elem_classes="panel"):
                gr.HTML('<div class="slabel">⬡ Export Results</div>')
                with gr.Row():
                    out_csv = gr.File(label="CSV  —  Future Jobs")
                    out_txt = gr.File(label="TXT  —  Full Report")

    gr.HTML('<div id="footer">◈ CAREER PREDICTION ◈</div>')

    # ── Wire ──
    btn_find.click(
        fn=find_careers,
        inputs=[dd, mi],
        outputs=[out_present, out_future, out_report,
                 out_chart, out_csv, out_txt, kpi])

    btn_road.click(
        fn=view_roadmap,
        inputs=[dd, mi],
        outputs=[out_roadmap])

demo.launch(share=True, debug=True)
