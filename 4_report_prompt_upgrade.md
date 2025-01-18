# 요청사항

- 너는 보고서 작성 전문가야
- 내가 제시하는 주제와 관련된 보고서를 작성해줘
- 보고서는 체계가 있어야 하고, □,○,- 순서로 내용이 세부적으로 들어가야 해
- 이 보고서에서 □는 총 3개를 제시해줘
- 각 □ 안에는 반드시 ○가 2개 이상 들어가야 함
- 각 ○안에는 -이 있어도 되고 없어도 됨
- □,○,- 모두 아래 응답 예시처럼 작성해줘
- 보고서를 작성한 후 아래 # 파이썬 코드를 이용해서 워드 문서를 만들어줘

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
        return 1.41 / 3.64  # ~0.387 cm per 글자
    elif malgun_font_size == 10:
        return 1.41 / 4.0
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
    run._element.rPr.rFonts.set(qn('w:eastAsia'), style.get("font_name", "바탕"))
    font.bold = style.get("bold", False)

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
            "hanging_indent": 0.75,
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

□ 생성형 AI는 다양한 분야에서 혁신을 이끌 수 있는 잠재력을 가지고 있으며, 이를 통해 생산성과 창의성을 동시에 증대시킬 수 있다.  
○ 먼저, 콘텐츠 제작 분야에서 생성형 AI를 활용하여 고품질의 창작물을 효율적으로 생산할 수 있다.  
- 예를 들어, 글쓰기, 음악 작곡, 영상 편집 등에서 AI가 자동으로 결과물을 생성하여 창작 시간을 단축시킬 수 있다.  
- 특히, 마케팅 콘텐츠의 경우 고객 데이터를 분석해 맞춤형 메시지를 자동으로 생성하는 데에도 유용하다.  

○ 또한, 교육 분야에서는 학습 자료를 자동 생성하거나 개인 맞춤형 학습 경험을 제공할 수 있다.  
- 예를 들어, 학습자의 수준에 맞는 문제나 강의 자료를 생성함으로써 맞춤형 학습을 지원한다.  
- AI가 교육용 게임이나 시뮬레이션을 생성하여 학습의 흥미를 유발하는 데에도 활용될 수 있다.  

□ 생성형 AI는 산업 분야에서도 작업 효율성을 향상시키는 데 기여할 수 있다.  
○ 제조업에서는 설계 자동화 및 시뮬레이션을 통해 제품 개발 시간을 단축할 수 있다.  
- 생성형 AI는 제품 디자인을 자동으로 생성하거나 기존 디자인을 최적화하는 데 사용될 수 있다.  
- 시뮬레이션 결과를 분석하여 제조 공정을 개선하는 데에도 활용될 수 있다.  

○ 서비스업에서는 고객 서비스 품질을 향상시키기 위한 도구로 활용 가능하다.  
- 예를 들어, 챗봇을 통해 고객 질문에 대한 실시간 대응이 가능하며, 자연어 처리 기술을 활용하여 고객 요구를 분석할 수 있다.  
- AI가 고객 데이터를 기반으로 맞춤형 서비스를 추천하거나 예약 시스템을 최적화할 수도 있다.  

□ 사회적 문제 해결 및 연구 개발 분야에서도 생성형 AI의 활용이 두드러진다.  
○ 보건 분야에서는 AI를 활용한 의료 데이터 분석 및 맞춤형 치료법 개발이 가능하다.  
- 생성형 AI는 새로운 의약품 설계나 질병 예측 모델 생성에 유용하다.  
- 예를 들어, 환자의 유전자 데이터를 분석하여 개인 맞춤형 치료법을 제안할 수 있다.  

○ 환경 분야에서는 데이터 분석과 예측을 통해 환경 보호와 지속 가능한 발전에 기여할 수 있다.  
- 기후 변화 데이터를 분석하여 예측 모델을 생성하고 이를 바탕으로 정책 수립에 도움을 줄 수 있다.  
- 생태계 보존을 위한 데이터 기반 시뮬레이션 및 솔루션을 제시할 수 있다.  
"""

output_path = "formatted_report_with_cm.docx"

# 실행
parse_and_format_docx(input_text, output_path)
print(f"Document saved to {output_path}")


```

# 보고서 주제 :

건강하게 운동을 하는 방법
