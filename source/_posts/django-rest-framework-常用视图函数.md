title: django-rest-framework 常用视图函数
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:10:00
---
### settings.py
```python
REST_FRAMEWORK = {
	# 设置分页
	'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
	'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning',
	'DEFAULT_PERMISSOIN_CLASSES':[
		'rest_framework.permissions.IsAdminUser',
	],
	# 每一页数量
	'PAGE_SIZE': 10
}
```

<!-- more -->

### models.py

```python
from django.db import models

SEX_CHOICES = (
	(1, '男'),
	(2, '女')
)


class Classes(models.Model):
	name = models.CharField(max_length=32)

	def __str__(self):
		return self.name


class Student(models.Model):
	name = models.CharField(max_length=16, null=False, verbose_name='学生姓名')
	sex = models.CharField(max_length=16, choices=SEX_CHOICES, default=1)
	classes = models.ForeignKey(Classes, on_delete=models.CASCADE)

	def __str__(self):
		return self.name


class Course(models.Model):
	name = models.CharField(max_length=32)

	def __str__(self):
		return self.name


class Score(models.Model):
	score = models.DecimalField(max_digits=5, decimal_places=2)
	student = models.ForeignKey(Student, on_delete=models.CASCADE)
	course = models.OneToOneField(Course, on_delete=models.CASCADE)

	def __str__(self):
		return self.score

```


### serializers.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-26 23:31

from rest_framework import serializers

from demo.models import Classes, Student, Course, Score


class ClassesSerializer(serializers.ModelSerializer):
	class Meta:
		model = Classes
		fields = ('id', 'name')


class StudentSerializer(serializers.ModelSerializer):
	class Meta:
		model = Student
		fields = ('id', 'name', 'sex', 'classes')


class CourseSerializer(serializers.ModelSerializer):
	class Meta:
		model = Course
		fields = ('id', 'name')


class ScoreSerializer(serializers.ModelSerializer):
	student = StudentSerializer(many=False, read_only=True)
	course = CourseSerializer(many=False, read_only=True)

	class Meta:
		model = Score
		fields = ('id', 'score', 'student', 'course')

```


### views.py
#### views_classes.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-26 23:57

from rest_framework import mixins
from rest_framework import generics

from demo.models import Classes
from demo.serializers import ClassesSerializer


class ClassesList(mixins.ListModelMixin,
				  mixins.CreateModelMixin,
				  generics.GenericAPIView):
	queryset = Classes.objects.all()
	serializer_class = ClassesSerializer

	def get(self, request, *args, **kwargs):
		return self.list(request, *args, **kwargs)

	def post(self, request, *args, **kwargs):
		return self.create(request, *args, **kwargs)


class ClassesDetail(mixins.RetrieveModelMixin,
					mixins.UpdateModelMixin,
					mixins.DestroyModelMixin,
					generics.GenericAPIView):
	queryset = Classes.objects.all()
	serializer_class = ClassesSerializer

	def get(self, request, *args, **kwargs):
		return self.retrieve(request, *args, **kwargs)

	def put(self, request, *args, **kwargs):
		return self.update(request, *args, **kwargs)

	def delete(self, request, *args, **kwargs):
		return self.destroy(request, *args, **kwargs)

```


#### views_course.py
```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.request import Request
from demo.models import Course
from demo.serializers import CourseSerializer


@api_view(['GET', 'POST'])
def course_list(request: Request):
	if request.method == 'GET':
		course = Course.objects.all()
		serializer = CourseSerializer(course, many=True)
		return Response(serializer.data)

	elif request.method == 'POST':
		serializer = CourseSerializer(data=request.data)
		if serializer.is_valid():
			serializer.save()

			return Response(serializer.data, status=status.HTTP_201_CREATED)

		return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'DELETE'])
def course_detail(request: Request, pk):
	try:
		course = Course.objects.get(pk=pk)
	except Course.DoesNotExist:
		return Response(status=status.HTTP_404_NOT_FOUND)

	if request.method == 'GET':
		serializer = CourseSerializer(course)
		return Response(serializer.data)

	elif request.method == 'PUT':
		serializer = CourseSerializer(course, data=request.data)
		if serializer.is_valid():
			serializer.save()

			return Response(serializer.data)

		return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

	elif request.method == 'DELETE':
		course.delete()
		return Response(status=status.HTTP_204_NO_CONTENT)

```


#### views_score.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-27 00:02


from rest_framework import generics

from demo.models import Score
from demo.serializers import ScoreSerializer


class ScoreList(generics.ListCreateAPIView):
	queryset = Score.objects.all()
	serializer_class = ScoreSerializer


class ScoreDetail(generics.RetrieveUpdateDestroyAPIView):
	queryset = Score.objects.all()
	serializer_class = ScoreSerializer

```


#### views_student.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-26 23:46

from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from demo.models import Student
from demo.serializers import StudentSerializer


class StudentList(APIView):

	def get(self, request, format=None):
		stu = Student.objects.all()
		serializer = StudentSerializer(stu, many=True)

		return Response(serializer.data)

	def post(self, request, format=None):
		serializer = StudentSerializer(data=request.data)
		if serializer.is_valid():
			serializer.save()

			return Response(serializer.data, status=status.HTTP_201_CREATED)

		return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class StudentDetail(APIView):

	def get_object(self, pk):
		try:
			return Student.objects.get(pk)

		except Student.DoesNotExist:
			raise Http404

	def get(self, request, pk, format=None):
		stu = self.get_object(pk)
		serializer = StudentSerializer(stu)
		return Response(serializer.data)

	def put(self, request, pk, format=None):
		stu = self.get_object(pk)
		serializer = StudentSerializer(stu, data=request.data)
		if serializer.is_valid():
			serializer.save()
			return Response(serializer.data)

		return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

	def delete(self, request, pk, format=None):

		stu = self.get_object(pk)
		stu.delete()

		return Response(status=status.HTTP_204_NO_CONTENT)

```


### urls.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-27 00:03


from django.urls import path, re_path

from demo import views_classes, views_course, views_score, views_student

urlpatterns = [
	path('cls/', views_classes.ClassesList.as_view()),
	re_path('cls/(?P<pk>[0-9]+)/', views_classes.ClassesDetail.as_view()),
]

urlpatterns += [
	path('stu/', views_student.StudentList.as_view()),
	re_path('stu/(?P<pk>[0-9]+)/', views_student.StudentDetail.as_view())
]

urlpatterns += [
	path('cou/', views_course.course_list),
	re_path('cou/(?P<pk>[0-9]+)/', views_course.course_detail),
]

urlpatterns += [
	path('sco/', views_score.ScoreList.as_view()),
	re_path('sco/(?P<pk>[0-9]+)', views_score.ScoreDetail.as_view())
]

```