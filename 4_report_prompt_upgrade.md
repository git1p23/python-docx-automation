# 요청사항

- 너는 보고서 작성 전문가야
- 내가 제시하는 주제와 관련된 보고서를 작성해줘
- 보고서는 체계가 있어야 하고, □,○,- 순서로 내용이 세부적으로 들어가야 해
- 이 보고서에서 □는 총 3개를 제시해줘
- 각 □ 안에는 반드시 ○가 2개 이상 들어가야 함
- 각 ○안에는 -이 있어도 되고 없어도 됨
- □,○,- 모두 아래 응답 예시처럼 작성해줘
- 먼저 보고서를 작성해줘
- 그리고 반드시 code interpreter로 아래 # 파이썬 코드를 실행해서 워드 문서파일을 생성해줘 (코드를 주지말고 반드시 직접 실행해줘)

# 응답 예시

□ 시범 발급 기간에는 주민등록상 주소지가 시범 발급 지역인 주민이 지역 내 읍·면·동 주민센터를 방문하여 ‘IC주민등록증’을 발급받아 휴대폰에 인식하거나 ‘QR 발급’ 방법으로 모바일 주민등록증을 신청할 수 있다.
○ 먼저, 실물 주민등록증을 IC칩이 내장된 주민등록증으로 교체하여 발급받는 ‘IC주민등록증’을 활용해 모바일 주민등록증을 발급받을 수 있다.

- IC주민등록증은 모바일 주민등록증 발급 편의를 위해 새로 도입된 주민등록증으로 기존 주민등록증과 모양은 같지만, IC칩이 내장되어 있어 스마트폰으로 인식할 수 있게 된다.
- 휴대전화를 바꾸거나 앱 삭제시 IC주민등록증만 있으면 주민센터를 방문하지 않고도 모바일 주민등록증을 재발급받을 수 있다.
- 다만, IC주민등록증 발급 시에는 1만 원의 비용이 소요된다.


# 파이썬 코드

```python
from docx import Document
from docx.shared import Pt
from docx.oxml.ns import qn
from docx.shared import Cm
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT


def calculate_char_width_multiplier(malgun_font_size=11):
    """
    워드문서 표준스타일에 따른 글자 / cm 비율
    표준스타일이 맑은고딕 11인 경우, 글자 1자 = 0.387cm
    """
    if malgun_font_size == 11:
        return 1.41 / 3.64  # 약 0.387 cm / 자
    elif malgun_font_size == 10:
        return 1.41 / 4.0  # 약 0.3525 cm / 자
    elif malgun_font_size == 12:
        return 1.41 / 3.33
    else:
        return 1.41 / 4.0


def format_paragraph(doc, text, style):
    """
    문단 스타일 설정
    """
    paragraph = doc.add_paragraph()
    run = paragraph.add_run(text)

    # 글꼴 설정
    font = run.font
    font.size = Pt(style.get("font_size", 12))
    font.name = style.get("font_name", "바탕")
    font.bold = style.get("bold", False)
    run._element.rPr.rFonts.set(qn('w:eastAsia'), style.get("font_name", "바탕")) # 챗GPT적용을 위해 꼭 필요
    paragraph_format = paragraph.paragraph_format

    # 글자 폭 계산 : 1자당 0.387cm 가정
    char_width_multiplier = calculate_char_width_multiplier()

    # 들여쓰기 설정
    char_indent_chars = style.get("char_indent", 0)
    hanging_indent_chars = style.get("hanging_indent", 0)

    paragraph_format.left_indent = Cm(char_width_multiplier * (char_indent_chars + hanging_indent_chars))
    paragraph_format.first_line_indent = Cm(-char_width_multiplier * hanging_indent_chars)

    # 문단 위 여백 설정
    paragraph_format.space_before = Pt(style.get("space_before", 0))

    # 줄 간격 설정
    paragraph_format.line_spacing = style.get("line_spacing", 1.5)

    # 정렬 설정
    paragraph.alignment = style.get("alignment", WD_PARAGRAPH_ALIGNMENT.LEFT)

    return paragraph


def set_margins(doc, top, bottom, left, right):
    """
    문서 여백 설정
    """
    section = doc.sections[0]
    section.top_margin = Cm(top)
    section.bottom_margin = Cm(bottom)
    section.left_margin = Cm(left)
    section.right_margin = Cm(right)


def parse_and_format_docx(input_text, output_path):
    """
    텍스트 파싱 및 Word 문서 생성
    """
    doc = Document()

    # 문서 여백 설정
    set_margins(doc, top=1.5, bottom=1.5, left=2.0, right=2.0)

    # 스타일 정의
    styles = {
        "title": {
            "font_name": "바탕",
            "font_size": 20,
            "bold": True,
            "space_before": 18,
            "alignment": WD_PARAGRAPH_ALIGNMENT.CENTER
        },
        "level_1": {
            "font_name": "바탕",
            "font_size": 14,
            "bold": True,
            "space_before": 18,
            "hanging_indent": 1.5,
        },
        "level_2": {
            "font_name": "바탕",
            "font_size": 14,
            "char_indent": 0.5,
            "hanging_indent": 1.5,
            "space_before": 12
        },
        "level_3": {
            "font_name": "바탕",
            "font_size": 14,
            "char_indent": 1.5,
            "hanging_indent": 1,
            "space_before": 6
        },
        "default": {
            "font_name": "바탕",
            "font_size": 14
        }
    }

    # 텍스트 파싱
    lines = input_text.split("\n")

    for line in lines:
        line = line.strip()
        if not line:
            continue

        # 스타일별 문단 생성
        if line.startswith("### "):
            format_paragraph(doc, line[4:], styles["title"])
        elif line.startswith("□"):
            format_paragraph(doc, line.strip(), styles["level_1"])
        elif line.startswith("○"):
            format_paragraph(doc, line.strip(), styles["level_2"])
        elif line.startswith("-"):
            format_paragraph(doc, line.strip(), styles["level_3"])
        else:
            format_paragraph(doc, line, styles["default"])

    # 파일 저장
    doc.save(output_path)


# 입력 텍스트
input_text = """### 생성형 AI의 활용방안

□ 생성형 AI는 콘텐츠 제작에서 생산성을 높이고 창의적인 결과물을 얻는 데 크게 기여할 수 있다.  
○ 먼저, 생성형 AI를 활용하면 글쓰기, 영상 제작, 음악 작곡 등 다양한 콘텐츠를 효율적으로 제작할 수 있다.  
- 예를 들어, 기업은 마케팅 자료나 광고 문구를 생성형 AI로 작성해 시간과 비용을 절약할 수 있다.  
- 개인 창작자는 음악이나 영상 편집 과정에서 생성형 AI의 도움을 받아 독창적인 작품을 빠르게 완성할 수 있다.  

○ 또한, 생성형 AI는 사용자 맞춤형 콘텐츠 제작에 유용하다.  
- 생성형 AI는 사용자의 선호도와 요구사항을 분석해 개인화된 콘텐츠를 제공할 수 있다.  
- 예를 들어, 온라인 교육 플랫폼은 학습자의 학습 수준과 관심사에 따라 맞춤형 교육 콘텐츠를 생성할 수 있다.  

□ 생성형 AI는 의료 및 헬스케어 분야에서도 중요한 역할을 할 수 있다.  
○ 생성형 AI를 통해 의료 진단 및 치료 계획 수립 과정에서 효율성과 정확성을 높일 수 있다.  
- AI가 의료 데이터를 분석해 진단에 필요한 정보와 치료 옵션을 제시할 수 있다.  
- 예를 들어, 환자의 유전자 정보를 기반으로 개인화된 치료 방안을 제안할 수 있다.  

○ 헬스케어 서비스에서도 생성형 AI를 활용해 환자와의 소통을 강화할 수 있다.  
- AI 챗봇은 환자의 질문에 실시간으로 응답하며, 기본적인 건강 정보를 제공할 수 있다.  
- 예를 들어, 복약 안내나 건강 관리에 필요한 정보를 환자 맞춤형으로 전달할 수 있다.  

□ 생성형 AI는 교육과 학습 효율을 향상시키는 데에도 기여할 수 있다.  
○ 생성형 AI는 학습 자료를 자동으로 생성하고, 학습자의 개별 수준에 맞춘 학습 경험을 제공할 수 있다.  
- 예를 들어, 생성형 AI는 교사가 입력한 간단한 개념을 기반으로 다양한 사례와 문제를 생성해 학습 자료를 풍성하게 만들 수 있다.  
- 또한, 학생의 이해도를 실시간으로 분석해 필요한 추가 자료를 제시할 수 있다.  

○ 원격 학습 환경에서도 생성형 AI는 효과적인 학습 지원 도구로 활용될 수 있다.  
- AI 기반 가상 교사는 학생들에게 실시간 피드백과 동기부여를 제공할 수 있다.  
- 또한, 외국어 학습에서는 대화형 AI를 활용해 실질적인 언어 사용 경험을 쌓게 할 수 있다.  
"""

output_path = "final_report.docx"

# 실행
parse_and_format_docx(input_text, output_path)
print(f"Document saved to {output_path}")
```

# 보고서 주제 :

양자 컴퓨팅의 미래
