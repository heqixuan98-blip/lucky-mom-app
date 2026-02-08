# lucky-mom-appimport streamlit as st
import time
import random
import json
import os
import imageio.v2 as imageio
from datetime import datetime

# ================= 1. 页面基础配置 =================

st.set_page_config(
    page_title="好孕签 · 您的孕期守护",
    page_icon="🏮",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ================= 2. 核心 UI 美化 (CSS) =================

st.markdown("""
    <style>
    /* --- 全局背景与字体 --- */
    .stApp {
        background-color: #fdfbf7; /* 暖米色背景 */
        background-image: linear-gradient(to bottom, #fff0f5 0%, #fdfbf7 100%);
        font-family: "PingFang SC", "Microsoft YaHei", sans-serif;
    }

    /* --- 侧边栏 --- */
    section[data-testid="stSidebar"] {
        background-color: #ffffff;
        border-right: 1px solid #f0f0f0;
    }
    
    /* --- 按钮大变身 (小红书红) --- */
    .stButton>button {
        background: linear-gradient(45deg, #ff2442, #ff5e62);
        color: white !important;
        border-radius: 25px;
        border: none;
        height: 50px;
        font-size: 16px;
        font-weight: 600;
        box-shadow: 0 4px 10px rgba(255, 36, 66, 0.3);
        transition: all 0.3s ease;
        width: 100%;
    }
    .stButton>button:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 15px rgba(255, 36, 66, 0.4);
    }

    /* --- 通用卡片容器 --- */
    .card {
        background-color: rgba(255, 255, 255, 0.9);
        padding: 25px;
        border-radius: 16px;
        box-shadow: 0 8px 20px rgba(0,0,0,0.05);
        margin-bottom: 20px;
        border: 1px solid #fff5f5;
        text-align: center;
        transition: transform 0.3s;
    }
    .card:hover {
        transform: translateY(-3px);
    }

    /* --- 结果卡片特别样式 --- */
    .result-box-blue {
        background: linear-gradient(135deg, #e3f2fd 0%, #ffffff 100%);
        border: 2px solid #bbdefb;
        border-radius: 15px;
        padding: 20px;
        color: #1565c0;
    }
    .result-box-pink {
        background: linear-gradient(135deg, #fce4ec 0%, #ffffff 100%);
        border: 2px solid #f8bbd0;
        border-radius: 15px;
        padding: 20px;
        color: #c2185b;
    }

    /* --- 底部文字 --- */
    .sub-text { font-size: 14px; color: #888; margin-top: 10px; }
    
    .footer {
        margin-top: 50px;
        padding-top: 20px;
        border-top: 1px dashed #ddd;
        text-align: center;
        color: #aaa;
        font-size: 12px;
    }
    </style>
    """, unsafe_allow_html=True)

# ================= 3. 数据与工具函数 =================

# 存储路径
PHOTO_DIR = "user_photos"
CAPSULE_FILE = "baby_letters.json"
if not os.path.exists(PHOTO_DIR): os.makedirs(PHOTO_DIR)

# 清宫图数据 (演示用)
QING_GONG_DATA = {
    18: [0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    19: [1, 0, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0],
    20: [0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1],
    21: [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    22: [0, 1, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0],
    23: [1, 1, 0, 1, 1, 0, 1, 0, 1, 1, 1, 0],
    24: [1, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0],
    25: [0, 1, 1, 0, 0, 1, 0, 1, 1, 1, 1, 1],
    26: [1, 0, 1, 0, 0, 1, 0, 1, 0, 0, 0, 0],
    27: [0, 1, 0, 1, 0, 0, 1, 1, 1, 1, 0, 1],
    28: [1, 0, 1, 0, 0, 0, 1, 1, 1, 1, 0, 0],
    29: [0, 1, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0],
    30: [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1],
}

# 吉祥话文案
LUCKY_MESSAGES = {
    "Blue": [
        {"title": "麒麟送子", "desc": "身形利落，气宇轩昂。古人云‘身带利器’，寓意小家伙精力充沛，未来定是护家小骑士。"},
        {"title": "如日方升", "desc": "肚型尖凸，线条硬朗。这不仅是阳刚之兆，更是家运昌隆的象征。"},
    ],
    "Pink": [
        {"title": "明珠入掌", "desc": "肚型圆润，宛若满月。这种‘藏风聚气’之相，预示着一位聪慧温婉的小棉袄。"},
        {"title": "花容月貌", "desc": "线条柔和，身形富贵。此乃‘凤落梧桐’之兆，宝宝定是才情兼备，福泽深厚。"},
    ]
}

def save_letter(content):
    if not os.path.exists(CAPSULE_FILE):
        with open(CAPSULE_FILE, 'w', encoding='utf-8') as f: json.dump([], f)
    with open(CAPSULE_FILE, 'r', encoding='utf-8') as f: letters = json.load(f)
    letters.append({"date": datetime.now().strftime("%Y-%m-%d"), "content": content})
    with open(CAPSULE_FILE, 'w', encoding='utf-8') as f: json.dump(letters, f)

def load_letters():
    if not os.path.exists(CAPSULE_FILE): return []
    with open(CAPSULE_FILE, 'r', encoding='utf-8') as f: return json.load(f)

def create_gif():
    images = []
    files = sorted(os.listdir(PHOTO_DIR))
    if not files: return None
    gif_path = "my_pregnancy_vlog.gif"
    with imageio.get_writer(gif_path, mode='I', duration=0.8, loop=0) as writer:
        for filename in files:
            if filename.endswith(('.png', '.jpg', '.jpeg')):
                image = imageio.imread(os.path.join(PHOTO_DIR, filename))
                writer.append_data(image)
    return gif_path

# ================= 4. 侧边栏导航 =================

with st.sidebar:
    st.image("https://cdn-icons-png.flaticon.com/512/3064/3064032.png", width=80)
    st.title("🧧 好孕签")
    st.markdown("**新中式·好运助手**")
    
    menu = st.radio(
        "功能菜单",
        ["🤰 肚型祈福 & 起名", "📜 清宫图测算", "📅 每日孕期黄历", "🎞️ 孕期时光机", "📸 宝宝长相/爸爸PK", "💌 给宝宝的信"],
    )
    
    st.markdown("---")
    st.warning("⚠️ **免责声明**：\n本应用所有结果基于中国传统民俗与娱乐算法，**无医学依据**。请勿作为性别判断标准。")

# ================= 5. 功能模块实现 =================

# --- 模块1: 肚型祈福 & 起名 ---
if menu == "🤰 肚型祈福 & 起名":
    st.header("🤰 肚型看寓意 & AI起名")
    st.caption("上传照片，查看民间说法，获取吉祥好名")

    uploaded_file = st.file_uploader("请拍摄或上传孕肚照", type=['jpg', 'png'])
    
    if uploaded_file:
        col1, col2 = st.columns([1, 2])
        with col1:
            st.image(uploaded_file, width=200, caption="已上传")
        with col2:
            if st.button("🔮 开始祈福测算"):
                with st.spinner('AI 正在翻阅《周易》...'):
                    time.sleep(1.5)
                    pred = random.choice(["Blue", "Pink"])
                    msg = random.choice(LUCKY_MESSAGES[pred])
                    st.session_state['pred'] = pred
                    st.session_state['analyzed'] = True
                    
                    # 使用新的卡片样式
                    css_class = "result-box-blue" if pred == "Blue" else "result-box-pink"
                    bg_icon = "🌊" if pred == "Blue" else "🌸"
                    icon = "🤴" if pred == "Blue" else "👸"
                    
                    st.markdown(f"""
                    <div class="card {css_class}">
                        <div style="font-size: 40px; margin-bottom: 10px;">{icon}</div>
                        <h2 style="margin: 0;">{msg['title']}</h2>
                        <div style="width: 50px; height: 2px; background-color: rgba(0,0,0,0.1); margin: 15px auto;"></div>
                        <p style="font-size: 18px; line-height: 1.6; font-weight: 500;">
                            {msg['desc']}
                        </p>
                        <div class="sub-text">
                            {bg_icon} 民间寓意 · 仅供娱乐 {bg_icon}
                        </div>
                    </div>
                    """, unsafe_allow_html=True)
                    st.balloons()

    # 起名部分 (注意缩进修正)
    if st.session_state.get('analyzed'):
        st.markdown("---")
        st.markdown("### ✨ 既然有好兆头，赐个名字吧？")
        with st.expander("📝 点击展开起名大师", expanded=True):
            surname = st.text_input("请输入宝宝姓氏", placeholder="例如：林")
            style = st.radio("选择风格", ["诗经楚辞 (古风)", "唐诗宋词 (唯美)", "现代寓意 (洋气)"])
            
            if st.button("💡 AI 生成美名"):
                if not surname:
                    st.toast("请先输入姓氏哦！")
                else:
                    st.success(f"为【{surname}】家宝宝生成的祈福美名：")
                    names = []
                    if st.session_state['pred'] == "Blue":
                        names = [
                            {"n": "星河", "s": "维天有汉，监亦有光", "m": "胸怀宽广"},
                            {"n": "景行", "s": "高山仰止，景行行止", "m": "品德高尚"},
                            {"n": "云帆", "s": "直挂云帆济沧海", "m": "志向远大"}
                        ]
                    else:
                        names = [
                            {"n": "静姝", "s": "静女其姝，俟我于城隅", "m": "娴静美好"},
                            {"n": "清婉", "s": "有美一人，清扬婉兮", "m": "气质高雅"},
                            {"n": "若初", "s": "人生若只如初见", "m": "初心不改"}
                        ]
                    
                    cols = st.columns(3)
                    for i, item in enumerate(names):
                        with cols[i]:
                            st.markdown(f"""
                            <div class="card">
                                <h3>{surname}{item['n']}</h3>
                                <p style="font-size:12px; color:#666">📜 {item['s']}</p>
                                <p style="font-size:12px; color:#ff2442">✨ {item['m']}</p>
                            </div>
                            """, unsafe_allow_html=True)

# --- 模块2: 清宫图测算 ---
elif menu == "📜 清宫图测算":
    st.header("📜 皇室秘传 · 清宫图")
    st.markdown("源自清代宫廷的数字游戏，看看老祖宗的算法准不准？")
    
    col1, col2 = st.columns(2)
    with col1:
        birth = st.date_input("👩 妈妈出生日期", min_value=datetime(1980,1,1), max_value=datetime(2005,12,31))
    with col2:
        conc = st.date_input("❤️ 受孕大概日期", min_value=datetime(2023,1,1))
        
    if st.button("🧮 立即查表"):
        age = conc.year - birth.year + 1 # 简易虚岁
        month = conc.month # 简易农历月
        
        st.markdown("---")
        if 18 <= age <= 30: 
            res = QING_GONG_DATA.get(age, [0]*12)[month-1]
            txt = "小皇子 (阳)" if res else "小格格 (阴)"
            bg_color = "#e3f2fd" if res else "#fce4ec"
            
            st.markdown(f"""
            <div class="card" style="background-color:{bg_color}">
                <h3>您的虚岁：{age} 岁 | 受孕月份：{month} 月</h3>
                <h1 style="color:#333; margin:20px 0">{txt}</h1>
                <p>根据《清宫珍藏生男育女表》推演</p>
            </div>
            """, unsafe_allow_html=True)
        else:
            st.info("💡 演示版仅录入 18-30 岁数据。")

# --- 模块3: 每日黄历 ---
elif menu == "📅 每日孕期黄历":
    st.header("📅 今日·孕妈运势")
    today = datetime.now().strftime("%Y年%m月%d日")
    st.subheader(today)
    
    random.seed(today)
    yi = random.choice(["听莫扎特", "吃红苹果", "对镜自夸", "散步30分钟", "整理宝宝衣服"])
    ji = random.choice(["熬夜追剧", "看恐怖片", "喝浓茶", "生气", "搬重物"])
    tip = random.choice(["今日宝宝听觉发育敏感，给他讲个笑话吧。", "今天适合给宝宝起乳名哦。", "记得补充叶酸和维生素。"])
    
    col1, col2 = st.columns(2)
    with col1:
        st.success(f"**宜**：{yi}")
    with col2:
        st.error(f"**忌**：{ji}")
        
    st.info(f"💡 **今日胎教小贴士**：{tip}")

# --- 模块4: 孕期时光机 ---
elif menu == "🎞️ 孕期时光机":
    st.header("🎞️ 孕期时光机")
    st.caption("每周打卡，最后生成一张动态变化图，见证生命奇迹！")
    
    tab1, tab2 = st.tabs(["📸 拍照打卡", "🎬 生成影片"])
    
    with tab1:
        week = st.slider("当前孕周", 1, 40, 12)
        pic = st.file_uploader(f"上传第 {week} 周照片", type=['jpg', 'png'])
        if pic and st.button("✅ 存入相册"):
            fname = f"{week:02d}周_{int(time.time())}.jpg"
            with open(os.path.join(PHOTO_DIR, fname), "wb") as f:
                f.write(pic.getbuffer())
            st.success("保存成功！坚持记录哦！")
            
        st.markdown("#### 最近打卡")
        files = sorted(os.listdir(PHOTO_DIR))
        if files:
            cols = st.columns(4)
            for i, f in enumerate(files[-4:]):
                cols[i].image(os.path.join(PHOTO_DIR, f), caption=f.split('_')[0])
                
    with tab2:
        if st.button("✨ 一键生成 GIF 动图"):
            with st.spinner("正在拼接时光碎片..."):
                gif = create_gif()
                if gif:
                    st.image(gif, caption="我的孕期变化", width=400)
                    with open(gif, "rb") as f:
                        st.download_button("📥 下载动图", f, file_name="my_pregnancy.gif")
                else:
                    st.warning("请先上传至少一张照片哦！")

# --- 模块5: 娱乐功能 ---
elif menu == "📸 宝宝长相/爸爸PK":
    st.header("📸 娱乐工坊")
    mode = st.radio("选择玩法", ["👶 AI预测宝宝长相", "🍺 准爸大肚PK"])
    
    if mode == "👶 AI预测宝宝长相":
        st.info("上传父母照片，AI融合生成宝宝样貌 (需接入GAN模型，此处为演示)")
        c1, c2 = st.columns(2)
        c1.file_uploader("妈妈照片")
        c2.file_uploader("爸爸照片")
        if st.button("🧩 开始融合"):
            time.sleep(2)
            st.image("https://images.unsplash.com/photo-1519689680058-324335c77eba?w=500", width=300, caption="预测结果：大眼睛像妈妈！")
            
    else:
        st.info("看看爸爸的肚子相当于怀孕几周？")
        dad_pic = st.file_uploader("上传爸爸大肚照")
        if dad_pic and st.button("🔍 鉴定"):
            res = random.choice(["双胞胎大西瓜 🍉", "陈年普洱茶肚 🍵", "九九归一腹肌 🧘‍♂️"])
            st.markdown(f"<div class='card'><h1>{res}</h1><p>恭喜爸爸，卸货遥遥无期！</p></div>", unsafe_allow_html=True)

# --- 模块6: 给宝宝的信 ---
elif menu == "💌 给宝宝的信":
    st.header("💌 存下一封跨越时空的信")
    
    with st.form("letter"):
        txt = st.text_area("亲爱的宝宝，我想对你说...", height=150)
        if st.form_submit_button("📪 封存信件"):
            save_letter(txt)
            st.success("信件已封存！")
            
    st.markdown("---")
    st.subheader("🗃️ 时光胶囊")
    letters = load_letters()
    if letters:
        for l in letters:
            with st.expander(f"✉️ {l['date']}"):
                st.write(l['content'])
    else:
        st.caption("还没有写过信，快来写第一封吧！")

# ================= 6. 底部合规声明 =================
st.markdown("""
    <div class="footer">
        <p>好孕签 App | 仅供民俗娱乐与生活记录</p>
        <p>隐私政策：您的照片仅用于本地分析，不会上传至第三方服务器。</p>
        <p>免责声明：本应用所有测试结果均为民俗说法，无医学依据，不作为诊断参考。</p>
    </div>
""", unsafe_allow_html=True)
