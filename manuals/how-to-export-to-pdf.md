There are a few ways to export data to a PDF file using Django. All of them requires a third-party library so to generate the file itself. First I will show how to return a PDF response, which can also be used if you are just serving an existing PDF file. Then I will show how to use ReportLab and WeasyPrint.

Writing a PDF to Response
In the example below I’m using the Django’s FileSystemStorage class. It will also work if you simply use open() instead. The FileSystemStorage sets the base_url to the project’s MEDIA_ROOT.

from django.core.files.storage import FileSystemStorage
from django.http import HttpResponse, HttpResponseNotFound

def pdf_view(request):
    fs = FileSystemStorage()
    filename = 'mypdf.pdf'
    if fs.exists(filename):
        with fs.open(filename) as pdf:
            response = HttpResponse(pdf, content_type='application/pdf')
            response['Content-Disposition'] = 'attachment; filename="mypdf.pdf"'
            return response
    else:
        return HttpResponseNotFound('The requested pdf was not found in our server.')
This way the user will be prompted with the browser’s open/save file. If you want to display the PDF in the browser you can change the Content-Disposition to:

response['Content-Disposition'] = 'inline; filename="mypdf.pdf"'
Using ReportLab
Installation
pip install reportlab
It relies on Pillow, which is a third-party Python Image Library. Sometimes it is a pain to get it installed. If you are running into problems, please refer to the Pillow’s installation guide.

Usage
A “Hello, World” from Django docs:

from io import BytesIO
from reportlab.pdfgen import canvas
from django.http import HttpResponse

def write_pdf_view(request):
    response = HttpResponse(content_type='application/pdf')
    response['Content-Disposition'] = 'inline; filename="mypdf.pdf"'

    buffer = BytesIO()
    p = canvas.Canvas(buffer)

    # Start writing the PDF here
    p.drawString(100, 100, 'Hello world.')
    # End writing

    p.showPage()
    p.save()

    pdf = buffer.getvalue()
    buffer.close()
    response.write(pdf)

    return response
Below an example of writing a PDF using SimpleDocTemplate and Paragraph:

from django.core.files.storage import FileSystemStorage
from django.http import HttpResponse

from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib.units import inch

def write_pdf_view(request):
    doc = SimpleDocTemplate("/tmp/somefilename.pdf")
    styles = getSampleStyleSheet()
    Story = [Spacer(1,2*inch)]
    style = styles["Normal"]
    for i in range(100):
       bogustext = ("This is Paragraph number %s.  " % i) * 20
       p = Paragraph(bogustext, style)
       Story.append(p)
       Story.append(Spacer(1,0.2*inch))
    doc.build(Story)

    fs = FileSystemStorage("/tmp")
    with fs.open("somefilename.pdf") as pdf:
        response = HttpResponse(pdf, content_type='application/pdf')
        response['Content-Disposition'] = 'attachment; filename="somefilename.pdf"'
        return response

    return response
ReportLab is a great library. It is very versatile. But there are a lot of configurations and settings to get the printing right. So make sure you really want to go down that road.

Refer to the ReportLab Userguide for more references about the functions, classes and all the available resources.

Using WeasyPrint
Installation
pip install WeasyPrint
Downside: that huge list of dependency. If you were lucky enough to get it installed smoothly, move forward. If that’s not the case, please refer to the official WeasyPrint installation guide.

Usage
A good thing about WeasyPrint is that you can convert a HTML document to a PDF. So you can create a regular Django template, print and format all the contents and then pass it to the WeasyPrint library to do the job of creating the pdf.

pdf_template.html

<html>
<head>
  <title>Test</title>
  <style>
    body {
      background-color: #f7f7f7;
    }
  </style>
</head>
<body>
  <h1>Test Template</h1>
  {% for paragraph in paragraphs %}
    <p>{{ paragraph }}</p>
  {% endfor %}
</body>
</html>
views.py

from django.core.files.storage import FileSystemStorage
from django.http import HttpResponse
from django.template.loader import render_to_string

from weasyprint import HTML

def html_to_pdf_view(request):
    paragraphs = ['first paragraph', 'second paragraph', 'third paragraph']
    html_string = render_to_string('core/pdf_template.html', {'paragraphs': paragraphs})

    html = HTML(string=html_string)
    html.write_pdf(target='/tmp/mypdf.pdf');

    fs = FileSystemStorage('/tmp')
    with fs.open('mypdf.pdf') as pdf:
        response = HttpResponse(pdf, content_type='application/pdf')
        response['Content-Disposition'] = 'attachment; filename="mypdf.pdf"'
        return response

    return response
Output PDF File

PDF Output

You can also have a look on the official API reference.

