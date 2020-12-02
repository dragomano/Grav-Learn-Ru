---
title: Обзор
taxonomy:
    category: docs
---

Если вы обнаружите возможную проблему безопасности, связанную с Grav или одним из его расширений, отправьте электронное письмо основной команде по адресу [contact@getgrav.org](mailto:contact@getgrav.org), и мы решим её, как только возможно.

Проблемы не должны разглашаться публично, в том числе на GitHub, Discord или форуме Discourse, до тех пор, пока основная команда не получит возможность изучить их и связаться с соответствующими сторонами для решения. Кроме того, если проблема не является потенциальной угрозой для пользователей Grav, ее, вероятно, следует отправить [как проблему](https://github.com/getgrav/grav/blob/develop/CONTRIBUTING.md#bug-reports ) вместо этого. Если вы не уверены, свяжитесь с нами, и мы поможем вам выяснить, кому принадлежит отчет.

## Отправка отчета

Когда вы обнаружили потенциальную уязвимость в ядре Grav или в одном из его расширений, рекомендуется проявить должную осмотрительность и сообщить об этом:

1. Укажите **номера версий** Grav и всех установленных расширений, а также **к какому компоненту** относится проблема.
2. Опишите уязвимость **подробно и кратко**, чтобы меньше времени тратилось на поиск ее источника.
3. Запишите точные шаги, необходимые для **воспроизведения среды**, в которой возникает уязвимость: какие настройки установлены в `system.yaml`, какой контент создается и какие системные настройки применяются?
4. Если возможно, опишите источник уязвимости и способы **ее исправления**, чтобы разработчики могли как восстановить, так и защитить ее.

### Ответственное раскрытие информации

Grav следует модели _responsible disclosure_ для представления обнаруженных уязвимостей. Это означает, что как только проблема обнаружена, протестирована и успешно продемонстрирована, разработчику(разработчикам) должен быть предоставлен период времени для исправления уязвимости, прежде чем она будет публично раскрыта. Это связано с тем, что поиск и тестирование решений заявленных проблем требуют много времени и времени, а Grav — это проект с открытым исходным кодом, авторы которого не имеют неограниченного времени, чтобы посвятить ему. Поэтому рекомендуется также предложить, как решить проблему или исправить ее, если у вас есть знания соответствующего кода.

## Процесс разрешения проблемы

Если ваш отчет точен и воспроизводится новая проблема безопасности, основная группа устранит ее как можно скорее. Когда это будет сделано, проблема и ее решение будут включены в [публичный репозиторий отчетов](/security/reports). Вы будете зачислены по имени и с дополнительной ссылкой на ваш веб-сайт/профиль в социальной сети, но, если хотите, вы можете запросить указание псевдонима или «анонимного репортера».

Отчеты и проблемы остаются конфиденциальными, пока проблема не будет решена. В случае, если сопровождающий расширения не может своевременно решить проблему, расширение удаляется из метода Grav Package Manner до тех пор, пока она не будет решена.

## Поддерживаемые версии

Поддерживается только текущая версия Grav `major.minor`. Это означает, что патчи реализованы в `major.minor.patch`, но не регрессивно назад для более старых версий Grav. Важно поддерживать установку в актуальном состоянии, и многие изменения полезны, даже если они явно не нужны с точки зрения безопасности.

## Уровни риска

Есть пять уровней риска, связанных с Grav как программным обеспечением:

- Очень критично
- критический
- Умеренно критично
- Менее критичный
- Не критично

Они рассчитываются на основе «[Общей системы оценки неправильного использования](https://www.nist.gov/news-events/news/2012/07/software-features-and-inherent-risks-nists-guide-rating-software)» (CMSS) [Национального института стандартов и технологий](https://www.nist.gov/) (NIST). Из-за отсутствия легко доступного калькулятора для Grav используйте [RiskCalc](https://security.drupal.org/riskcalc) Drupal ([примечания](https://www.mydropwizard.com/blog/understanding-drupal-security-advisories-risk-calculator)).