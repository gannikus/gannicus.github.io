+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
+++

<!-- Example usage: responsive image shortcode -->
<!-- Put an image in the same folder as the page (page bundle) and use: -->
<!-- {{< responsive src="myphoto.jpg" alt="描述文本" >}} -->

