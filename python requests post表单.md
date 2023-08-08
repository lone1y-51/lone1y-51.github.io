- [[#最简单的post一个表单的方法|最简单的post一个表单的方法]]
- [[#post一个文件到服务器|post一个文件到服务器]]
- [[#post文件的同时携带其他参数|post文件的同时携带其他参数]]
- [[#源码分析|源码分析]]
	- [[#源码分析#延伸|延伸]]

## 最简单的post一个表单的方法
不需要指定`Content-Type`
```python
import requests
url = "http://xxxxxxxx"
data = {
    "a":"b"
}
requests.post(url, data=data)
```
如果指定了header也没有问题，可以正常使用
```python
import requests
url = "http://xxxxxxxx"
data = {
    "a":"b"
}
headers = {
    "Content-Type: application/x-www-form-urlencoded"
}
requests.post(url, data=data, headers=headers)
```
## post一个文件到服务器
```python
import requests
url = "http://xxxxxxxx"
files = {"file":open(file_path, 'r')}

requests.post(url, files=files)
```
这个时候`Content-Type`会被设置为`application/multipart-form`
当`Content-Type`是`multipart-form`时需要`boundary`来处理数据字段，上面这种写法`boundary`是有`requests`自己生成，且会拼接到`Content-Type`的后面。
这也就是说，如果post一个文件，并且手动设置`Content-Type`会导致服务端的不解析，类似下面这种用法：
```python
import requests
url = "http://xxxxxxxx"
files = {"file":open(file_path, 'r')}
headers = {
	"Content-Type: application/multipart-form"
}

requests.post(url, files=files, headers=headers)
```
## post文件的同时携带其他参数
```python
import requests
url = "http://xxxxxxxx"
files = {"file":open(file_path, 'r')}
data = {"a":"b"}

requests.post(url, files=files, data=data)
```
同理，这种方式也是不能主动设置`header`的。
这个时候`Content-Type`会被设置为`application/multipart-form`
## 源码分析
为什么`multipart-form`不能手动设置header呢
通过一步步查看源码，`reqeusts`包所有的请求最终都会进入到这样一段代码
```python
def request(method, url, **kwargs):
	with sessions.Session() as session:
        return session.request(method=method, url=url, **kwargs)
```
`session`的`request`方法如下
```python
def request(
        self,
        method,
        url,
        params=None,
        data=None,
        headers=None,
        cookies=None,
        files=None,
        auth=None,
        timeout=None,
        allow_redirects=True,
        proxies=None,
        hooks=None,
        stream=None,
        verify=None,
        cert=None,
        json=None,
    ):
        req = Request(
            method=method.upper(),
            url=url,
            headers=headers,
            files=files,
            data=data or {},
            json=json,
            params=params or {},
            auth=auth,
            cookies=cookies,
            hooks=hooks,
        )
        prep = self.prepare_request(req)

        proxies = proxies or {}

        settings = self.merge_environment_settings(
            prep.url, proxies, stream
        # 省略一些不必要的
```
继续深入`self.prepare_request`
```python
def prepare_request(self, request):
	## 省略一些不必要的
	p = PreparedRequest()
	p.prepare(
		method=request.method.upper(),
		url=request.url,
		files=request.files,
		data=request.data,
		json=request.json,
		headers=merge_setting(
			request.headers, self.headers, dict_class=CaseInsensitiveDict
		),
		params=merge_setting(request.params, self.params),
		auth=merge_setting(auth, self.auth),
		cookies=merged_cookies,
		hooks=merge_hooks(request.hooks, self.hooks),
	)
	return p
```
在继续深入`prepare`
```python
def prepare(
        self,
        method=None,
        url=None,
        headers=None,
        files=None,
        data=None,
        params=None,
        auth=None,
        cookies=None,
        hooks=None,
        json=None,
    ):
        """Prepares the entire request with the given parameters."""

        self.prepare_method(method)
        self.prepare_url(url, params)
        self.prepare_headers(headers)
        self.prepare_cookies(cookies)
        self.prepare_body(data, files, json)
        self.prepare_auth(auth, url)

        # Note that prepare_auth must be last to enable authentication schemes
        # such as OAuth to work on a fully prepared request.

        # This MUST go after prepare_auth. Authenticators could add a hook
        self.prepare_hooks(hooks)
```
这里可以发现`headers`的prepare在body之前，data/files/json在一个方法里进行的处理
进入`prepare_body`看下流程
```python
def prepare_body(self, data, files, json=None):
	"""Prepares the given HTTP body data."""

	# Check if file, fo, generator, iterator.
	# If not, run through normal process.

	# Nottin' on you.
	body = None
	content_type = None

	if not data and json is not None:
		# urllib3 requires a bytes-like body. Python 2's json.dumps
		# provides this natively, but Python 3 gives a Unicode string.
		content_type = "application/json"

		try:
			body = complexjson.dumps(json, allow_nan=False)
		except ValueError as ve:
			raise InvalidJSONError(ve, request=self)

		if not isinstance(body, bytes):
			body = body.encode("utf-8")

	is_stream = all(
		[
			hasattr(data, "__iter__"),
			not isinstance(data, (basestring, list, tuple, Mapping)),
		]
	)

	if is_stream:
		# 忽略这段代码
	else:
		# Multi-part file uploads.
		if files:
			(body, content_type) = self._encode_files(files, data)
		else:
			if data:
				body = self._encode_params(data)
				if isinstance(data, basestring) or hasattr(data, "read"):
					content_type = None
				else:
					content_type = "application/x-www-form-urlencoded"

		self.prepare_content_length(body)

		# Add content-type if it wasn't explicitly provided.
		if content_type and ("content-type" not in self.headers):
			self.headers["Content-Type"] = content_type

	self.body = body
```
这段代码已经比较清晰了
1. 没有`data`但是有`json`，会被认为是post的json数据，但是没有设置到`headers`里，所有需要json手动设置（不清楚为什么这么做，猜测是个bug?）
2. 当有`files`时会调用`_encode_files`方法返回body和content_type
3. 当没有`files`但是有`data`时，content_type会被设置成`application/x-www-form-urlencoded`
2，3步的content_type是否生效和`headers`中是否有`Content-Type`有关系，结合之前发现的prepare_header在prepare_body之前可以确定，传入的`Content-Type`优先级高于框架自己判断的
### 延伸
`_encode_files`代码
```python
def _encode_files(files, data):
        if not files:
            raise ValueError("Files must be provided.")
        elif isinstance(data, basestring):
            raise ValueError("Data must not be a string.")

        new_fields = []
        fields = to_key_val_list(data or {})
        files = to_key_val_list(files or {})

        for field, val in fields:
            if isinstance(val, basestring) or not hasattr(val, "__iter__"):
                val = [val]
            for v in val:
                if v is not None:
                    # Don't call str() on bytestrings: in Py3 it all goes wrong.
                    if not isinstance(v, bytes):
                        v = str(v)

                    new_fields.append(
                        (
                            field.decode("utf-8")
                            if isinstance(field, bytes)
                            else field,
                            v.encode("utf-8") if isinstance(v, str) else v,
                        )
                    )

        for (k, v) in files:
            # support for explicit filename
            ft = None
            fh = None
            if isinstance(v, (tuple, list)):
                if len(v) == 2:
                    fn, fp = v
                elif len(v) == 3:
                    fn, fp, ft = v
                else:
                    fn, fp, ft, fh = v
            else:
                fn = guess_filename(v) or k
                fp = v

            if isinstance(fp, (str, bytes, bytearray)):
                fdata = fp
            elif hasattr(fp, "read"):
                fdata = fp.read()
            elif fp is None:
                continue
            else:
                fdata = fp

            rf = RequestField(name=k, data=fdata, filename=fn, headers=fh)
            rf.make_multipart(content_type=ft)
            new_fields.append(rf)
        # 重点在这里，body和content_type
        body, content_type = encode_multipart_formdata(new_fields)

        return body, content_type
```
再继续往下看
```python
def encode_multipart_formdata(fields, boundary=None):
    body = BytesIO()
    if boundary is None:
        boundary = choose_boundary()

    for field in iter_field_objects(fields):
        body.write(b("--%s\r\n" % (boundary)))

        writer(body).write(field.render_headers())
        data = field.data

        if isinstance(data, int):
            data = str(data)  # Backwards compatibility

        if isinstance(data, six.text_type):
            writer(body).write(data)
        else:
            body.write(data)

        body.write(b"\r\n")

    body.write(b("--%s--\r\n" % (boundary)))

    content_type = str("multipart/form-data; boundary=%s" % boundary)

    return body.getvalue(), content_type
```
可以发现，boundary就是这里处理的，如果这个content_type被替换的话，就会导致body里的`boundary`和header里的对不上，从而服务端无法解析。