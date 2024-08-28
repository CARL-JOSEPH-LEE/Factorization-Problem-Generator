import tkinter as tk
from tkinter import ttk, messagebox
import sympy as sp
import random
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.pyplot as plt

# 难度等级对应的系数范围 (递进关系)
difficulty_levels = {
    "非常简单": (1, 2),
    "很简单": (3, 5),
    "简单": (6, 10),
    "一般": (11, 100),
    "困难": (101, 1000),
    "非常难": (1001, 10000),
    "噩梦": (10001, 100000),
    "地狱": (100001, 1000000)
}

# 生成题目计数器
question_counter = 0

# 当前生成的题目名称
current_question_name = ""

# 生成变量列表
def generate_variable_list(num_vars):
    if num_vars == 1:
        return [sp.symbols('x')]
    elif num_vars <= 4:
        variables = ['x', 'y', 'z', 'w'][:num_vars]  # 生成 'x', 'y', 'z', 'w' 中的前 num_vars 个变量
        return sp.symbols(' '.join(variables))       # 使用空格分隔符生成符号
    elif num_vars <= 52:
        variables = [chr(i) for i in range(97, 123)] + [chr(i) for i in range(65, 91)]  # a-z, A-Z
        return sp.symbols(' '.join(variables[:num_vars]))
    else:
        return sp.symbols(' '.join([f'x_{i}' for i in range(1, num_vars + 1)]))



# 生成不可分解的线性或二次因子
def generate_irreducible_factor(variables, coeff_range):
    """
    生成一个不可分解的一元一次多项式因子，所有变量都至少出现一次且系数不为0。

    :param variables: 变量列表
    :param coeff_range: 系数范围
    :return: 多项式因子
    """
    factor = 0
    for var in variables:
        coeff = random.randint(*coeff_range)
        while coeff == 0:  # 确保系数不为0
            coeff = random.randint(*coeff_range)
        factor += coeff * var

    # 添加一个非零的常数项
    constant_term = random.randint(*coeff_range)
    while constant_term == 0:
        constant_term = random.randint(*coeff_range)

    factor += constant_term

    return factor


# 生成可分解的多元多项式
def generate_factorable_polynomial(degree, num_vars, difficulty):
    variables = generate_variable_list(num_vars)
    min_coeff, max_coeff = difficulty
    polynomial = 1
    for _ in range(degree):
        factor = generate_irreducible_factor(variables, (min_coeff, max_coeff))
        polynomial *= factor

    return polynomial.expand()

# 保存多项式为文本文件
def save_expression_to_txt(expr, filename):
    with open(filename, 'w') as file:
        file.write(str(expr))

# 显示数学表达式，直接显示格式化后的表达式
def display_expression(expr, label, filename):
    try:
        # 增大图像尺寸，确保更复杂的表达式能显示出来
        fig, ax = plt.subplots(figsize=(10, 6))  # 图像尺寸调整为更大的10x6英寸

        # 使用 LaTeX 渲染表达式，自动换行处理
        # r"$\displaystyle {expr}$" 使用 LaTeX 的可显示样式，使表达式显示更自然
        ax.text(0.5, 0.5, f"${sp.latex(expr)}$", fontsize=18, ha='center', va='center', wrap=True)

        # 设置背景为白色
        ax.axis('off')
        fig.patch.set_facecolor('white')
        ax.set_facecolor('white')

        fig.savefig(filename, bbox_inches='tight', pad_inches=0.1, dpi=1000)

        # 将图像嵌入到 Tkinter 界面
        canvas = FigureCanvasTkAgg(fig, master=label)
        canvas.draw()
        canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)

        # 关闭图形以节省资源
        plt.close(fig)
    except Exception as e:
        messagebox.showerror("渲染错误", f"无法生成图像: {str(e)}")


def generate_question():
    global question_counter, current_question_name
    degree = int(degree_var.get())
    num_vars = int(var_count_var.get())
    difficulty = difficulty_levels[difficulty_var.get()]
    polynomial = generate_factorable_polynomial(degree, num_vars, difficulty)

    question_counter += 1
    current_question_name = f"{num_vars}元{degree}次多项式因式分解问题{question_counter}"
    txt_filename = f"{current_question_name}.txt"
    latex_filename = f"{current_question_name}.png"

    save_expression_to_txt(polynomial, txt_filename)

    for widget in question_label.winfo_children():
        widget.destroy()
    display_expression(polynomial, question_label, latex_filename)

    global current_polynomial
    current_polynomial = polynomial

    messagebox.showinfo("题目生成", f"题目已保存为 {txt_filename} 和 {latex_filename}")

# 查看答案
def show_answer():
    if current_polynomial is None:
        messagebox.showwarning("警告", "请先生成题目！")
    else:
        factored = sp.factor(current_polynomial)
        answer_filename = f"{current_question_name}答案.txt"
        latex_filename = f"{current_question_name}答案.png"

        save_expression_to_txt(factored, answer_filename)

        for widget in answer_label.winfo_children():
            widget.destroy()
        display_expression(factored, answer_label, latex_filename)

        messagebox.showinfo("答案生成", f"答案已保存为 {answer_filename} 和 {latex_filename}")

# 初始化窗口
root = tk.Tk()
root.title("因式分解题目生成器")
root.geometry("900x700")
root.configure(bg='#f7f7f7')

current_polynomial = None

# 使用ttk进行UI美化
style = ttk.Style()
style.configure('TButton', font=('Helvetica', 14), padding=10)
style.configure('TLabel', font=('Helvetica', 14))
style.configure('TFrame', background='#f7f7f7')

mainframe = ttk.Frame(root, padding="20 20 20 20", style='TFrame')
mainframe.pack(expand=True)

title_label = ttk.Label(mainframe, text="因式分解题目生成器", style='TLabel')
title_label.config(font=('Helvetica', 18, 'bold'), foreground='#007acc')
title_label.pack(pady=20)

ttk.Label(mainframe, text="请选择多项式的次数 (m):", style='TLabel').pack(pady=10)
degree_var = tk.StringVar(value="2")
degree_entry = ttk.Entry(mainframe, textvariable=degree_var, width=10, font=('Helvetica', 14))
degree_entry.pack(pady=10)

ttk.Label(mainframe, text="请选择多项式的元数 (n):", style='TLabel').pack(pady=10)
var_count_var = tk.StringVar(value="1")
var_count_entry = ttk.Entry(mainframe, textvariable=var_count_var, width=10, font=('Helvetica', 14))
var_count_entry.pack(pady=10)

ttk.Label(mainframe, text="请选择难度等级:", style='TLabel').pack(pady=10)
difficulty_var = tk.StringVar(value="非常简单")
difficulty_menu = ttk.OptionMenu(mainframe, difficulty_var, *difficulty_levels.keys())
difficulty_menu.pack(pady=10)

generate_button = ttk.Button(mainframe, text="生成题目", command=generate_question, style='TButton')
generate_button.pack(pady=20)

answer_button = ttk.Button(mainframe, text="查看答案", command=show_answer, style='TButton')
answer_button.pack(pady=20)

question_label = ttk.Frame(mainframe, style='TFrame')
question_label.pack(expand=True, fill=tk.BOTH, pady=10)

answer_label = ttk.Frame(mainframe, style='TFrame')
answer_label.pack(expand=True, fill=tk.BOTH, pady=10)

# 运行主循环
root.mainloop()