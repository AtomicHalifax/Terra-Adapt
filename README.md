```html
<div align="center">

<svg width="100%" viewBox="0 0 1200 430" xmlns="http://www.w3.org/2000/svg">

  <defs>
    <linearGradient id="bg" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#ffffff"/>
      <stop offset="100%" stop-color="#f4f7f5"/>
    </linearGradient>

    <linearGradient id="green" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#164e3b"/>
      <stop offset="100%" stop-color="#4f8f73"/>
    </linearGradient>

    <filter id="shadow" x="-20%" y="-20%" width="140%" height="140%">
      <feDropShadow dx="0" dy="12" stdDeviation="18"
                    flood-color="#18382d" flood-opacity="0.10"/>
    </filter>

    <clipPath id="earthClip">
      <circle cx="930" cy="205" r="145"/>
    </clipPath>
  </defs>

  <!-- Background -->
  <rect width="1200" height="430" rx="28" fill="url(#bg)"/>

  <!-- Editorial border -->
  <rect x="18" y="18" width="1164" height="394" rx="22"
        fill="none" stroke="#dfe6e2" stroke-width="1"/>

  <!-- Technical grid -->
  <g opacity="0.45" stroke="#dce5e0" stroke-width="1">
    <line x1="55" y1="65" x2="1145" y2="65"/>
    <line x1="55" y1="365" x2="1145" y2="365"/>
    <line x1="55" y1="65" x2="55" y2="365"/>
    <line x1="1145" y1="65" x2="1145" y2="365"/>
  </g>

  <!-- Small label -->
  <text x="70" y="92"
        font-family="Arial, Helvetica, sans-serif"
        font-size="13"
        font-weight="600"
        letter-spacing="3"
        fill="#5d6c65">
    SATELLITE VISION / DOMAIN ADAPTATION
  </text>

  <!-- Main title -->
  <text x="68" y="165"
        font-family="Arial, Helvetica, sans-serif"
        font-size="66"
        font-weight="800"
        letter-spacing="-2"
        fill="#111713">
    TERRA
  </text>

  <text x="68" y="225"
        font-family="Arial, Helvetica, sans-serif"
        font-size="66"
        font-weight="800"
        letter-spacing="-2"
        fill="#285f49">
    ADAPT
  </text>

  <!-- Subtitle -->
  <text x="72" y="264"
        font-family="Arial, Helvetica, sans-serif"
        font-size="17"
        fill="#4c5953">
    Adapting pretrained land-cover models
  </text>

  <text x="72" y="288"
        font-family="Arial, Helvetica, sans-serif"
        font-size="17"
        fill="#4c5953">
    to a new Sentinel-2 target domain.
  </text>

  <!-- Pipeline -->
  <g font-family="Arial, Helvetica, sans-serif"
     font-size="11"
     font-weight="700"
     letter-spacing="1">

    <rect x="70" y="322" width="92" height="31" rx="15"
          fill="#ffffff" stroke="#cfd9d4"/>
    <text x="116" y="342" text-anchor="middle" fill="#26342e">
      EUROSAT
    </text>

    <text x="174" y="343" fill="#7b8983">→</text>

    <rect x="190" y="322" width="92" height="31" rx="15"
          fill="#ffffff" stroke="#cfd9d4"/>
    <text x="236" y="342" text-anchor="middle" fill="#26342e">
      RESNET50
    </text>

    <text x="294" y="343" fill="#7b8983">→</text>

    <rect x="310" y="322" width="128" height="31" rx="15"
          fill="#285f49"/>
    <text x="374" y="342" text-anchor="middle" fill="#ffffff">
      ADAPTATION
    </text>

    <text x="450" y="343" fill="#7b8983">→</text>

    <rect x="466" y="322" width="112" height="31" rx="15"
          fill="#ffffff" stroke="#cfd9d4"/>
    <text x="522" y="342" text-anchor="middle" fill="#26342e">
      SENTINEL-2
    </text>
  </g>

  <!-- Earth / satellite visual -->
  <g filter="url(#shadow)">
    <circle cx="930" cy="205" r="145" fill="#eef4f0"/>
  </g>

  <g clip-path="url(#earthClip)">

    <!-- Earth base -->
    <circle cx="930" cy="205" r="145" fill="#dce9e2"/>

    <!-- Stylized satellite imagery -->
    <path d="M760 225
             C820 165 855 190 890 155
             C925 120 975 135 1005 168
             C1035 200 1090 188 1115 225
             L1115 370 L760 370 Z"
          fill="#9fbea9"/>

    <path d="M770 305
             C820 260 850 285 885 250
             C920 215 955 235 985 270
             C1010 300 1060 292 1105 260
             L1120 370 L760 370 Z"
          fill="#6f9b81"/>

    <!-- Water -->
    <path d="M790 120
             C845 150 860 175 850 205
             C840 235 800 240 780 215
             L760 120 Z"
          fill="#b9d7d2"/>

    <!-- Agricultural fields -->
    <g opacity="0.72" stroke="#d8e7dc" stroke-width="4">
      <line x1="895" y1="170" x2="855" y2="350"/>
      <line x1="925" y1="155" x2="900" y2="350"/>
      <line x1="955" y1="150" x2="945" y2="350"/>
      <line x1="985" y1="165" x2="990" y2="350"/>
      <line x1="1015" y1="185" x2="1035" y2="350"/>
    </g>

    <!-- Built area -->
    <g fill="#73827a">
      <rect x="1010" y="215" width="25" height="20"/>
      <rect x="1040" y="225" width="35" height="27"/>
      <rect x="1000" y="245" width="22" height="19"/>
      <rect x="1060" y="260" width="27" height="23"/>
    </g>

  </g>

  <!-- Earth outline -->
  <circle cx="930" cy="205" r="145"
          fill="none" stroke="#285f49" stroke-width="2"/>

  <!-- Orbit -->
  <ellipse cx="930" cy="205" rx="188" ry="63"
           fill="none" stroke="#8ba99a"
           stroke-width="1.5"
           transform="rotate(-18 930 205)"/>

  <!-- Satellite -->
  <g transform="translate(1058 112) rotate(-18)">
    <rect x="-20" y="-10" width="40" height="20"
          rx="3" fill="#26342e"/>
    <rect x="-52" y="-7" width="25" height="14"
          fill="#9ab8aa"/>
    <rect x="27" y="-7" width="25" height="14"
          fill="#9ab8aa"/>
    <line x1="0" y1="10" x2="0" y2="28"
          stroke="#26342e" stroke-width="2"/>
    <circle cx="0" cy="32" r="4" fill="#285f49"/>
  </g>

  <!-- Corner metadata -->
  <text x="1090" y="350"
        font-family="Arial, Helvetica, sans-serif"
        font-size="11"
        text-anchor="end"
        letter-spacing="2"
        fill="#68766f">
    WATER · TREES · CROPS · BUILT
  </text>

  <text x="1090" y="372"
        font-family="Arial, Helvetica, sans-serif"
        font-size="10"
        text-anchor="end"
        letter-spacing="1.5"
        fill="#89958f">
    SENTINEL-2 / DYNAMIC WORLD
  </text>

</svg>

</div>
```
