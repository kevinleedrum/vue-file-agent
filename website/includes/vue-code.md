
{% capture el_start %}
<div id="{{ include.name }}-mount">
  <{{ include.name }}></{{ include.name }}>
</div>
<script type="text/x-template" id="{{ include.name }}-template">
{% endcapture %}

{% capture cmp_start %}
var component = { template: '#{{ include.name }}-template',
{% endcapture %}

{% capture vue_mount %}
; new Vue({ components: {'{{ include.name }}': component}, el: '#{{ include.name }}-mount'});
</script>
{% endcapture %}

{% capture processed_code %}
{{ include.code | replace: "export default {", cmp_start, | replace: "<script>", "<script type='text/javascript'>" | replace: "</script>", vue_mount | replace: "<template>", el_start | replace: "</template>", "</script>" | replace: "<style>", "<style type='text/css'>" }}
{% endcapture %}


{% if include.result_only %}
  <div id="{{ include.name }}-wrapper">
    {{ processed_code }}
  </div>
{% else %}
  <div id="{{ include.name }}-wrapper">
    <div class="row">
      <div class="col-md-6">
        <blockquote>
          Code
          {% if include.codepen %}
            <a target="_blank" class="codepen-link" href="{{ include.codepen }}">Edit CodePen</a>
          {% endif %}
        </blockquote>
        {% highlight html %}{{ include.code | strip }}{% endhighlight %}
      </div>
      <div class="col-md-6">
        <blockquote>
          Result
          {% if include.codepen %}
            <a target="_blank" class="codepen-link" href="{{ include.codepen }}">Edit CodePen</a>
          {% endif %}
        </blockquote>
        {{ processed_code }}
      </div>
    </div>
  </div>
{% endif %}

<style type="text/css">
  #{{ include.name }}-wrapper blockquote {
    background: #EEE;
    color: #000;
    font-weight: bold;
    margin-bottom: 0;
    padding: 2px 10px;
  }
  #{{ include.name }}-wrapper .codepen-link {
    float: right;
  }
  #{{ include.name }}-wrapper .highlight pre {
    max-height: 300px;
  }
</style>