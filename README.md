# HLSL-Basic-Light-Model

**Основные компоненты освещения**
Стандартная модель освещения состоит из трех ключевых компонентов:

  * Ambient Light (Фоновое освещение) - равномерное освещение без направления, имитирующее рассеянный свет в окружающей среде
  * Diffuse Light (Рассеянный свет) - свет, равномерно отражающийся от поверхности в зависимости от угла падения
  * Specular Light (Зеркальный свет) - блики, возникающие при отражении света от поверхности

```glsl
float4 AmbientColor : Ambient = (0.25f, 0.25f, 0.25f, 1.0f);
float4 DiffuseColor : Diffuse = (1.0f, 1.0f, 1.0f, 1.0f); 
float4 SpecularColor : Specular = (1.0f, 1.0f, 1.0f, 1.0f);
```

**Принципы реализации**
1. Фоновое освещение (Ambient)
Самый простой компонент, который добавляет базовый уровень освещенности ко всем поверхностям независимо от их ориентации или положения относительно источников света.

2. Рассеянный свет (Diffuse)
Рассчитывается на основе:
 - Угла между нормалью поверхности и направлением к источнику света (закон Ламберта)
 - Цвета поверхности (из текстуры)
 - Цвета и интенсивности источника света

```glsl
Использует закон Ламберта:

float diffuselight = saturate(dot(N,L));
float4 Diffuse = diffuselight * DiffuseColor * ColorTexture * lightColor;
```

3. Зеркальные блики (Specular)
Зависит от:
  - Угла между направлением отраженного света и направлением к камере
  - "Глянцевости" поверхности (параметр Glossiness)
  - Интенсивности бликов (Specular Color)

```glsl
Модель Блинна-Фонга:

float H = normalize(L + V);
float NdotH = saturate(dot(N, H));
float SpecPower = pow(NdotH, Glossiness);
float4 Specular = SpecPower * SpecularColor * SpecularTexture;
```

**Технические аспекты**
Шейдер состоит:
  - Вершинный шейдер для преобразования координат и расчета векторов
  - Пиксельный шейдер для вычисления итогового цвета каждого пикселя
  - Поддержку текстур (диффузная, зеркальная, нормалей)
  - Настройки материалов (цвета, глянцевость)
  - Параметры источников света (положение, цвет)

```glsl
Вершинный шейдер

struct v2f {
  float3 lightVec : TEXCOORD1;
  float3 eyeVec : TEXCOORD2;
  float3 worldNormal : TEXCOORD3;
};

Пиксельный шейдер
hlsl
float4 ColorTexture = tex2D(diffuseMapSampler, In.texCoord.xy);
float4 SpecularTexture = tex2D(specularMapSampler, In.texCoord.xy);
float3 normal = tex2D(normalMapSampler, In.texCoord).xyz * 2.0 - 1;
```

Итоговое освещение комбинируется из всех компонентов:
```glsl
return (Ambient + Diffuse + Specular) * lightColor;
```
