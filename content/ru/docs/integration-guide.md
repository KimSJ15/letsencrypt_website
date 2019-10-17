---
title: Руководство по интеграции
slug: integration-guide
top_graphic: 1
date: 2016-08-08
lastmod: 2018-06-20
---

{{< lastmod >}}

Этот документ будет полезен хостинг-провайдерам, интеграторам и разработчикам клиентского ПО для Let's Encrypt.

# Планирование изменений

И Let's Encrypt, и технология Web PKI продолжают развиваться. Вам нужно быть готовыми к необходимым обновлениям всех сервисов, которые использует Let's Encrypt. Особое внимание необходимо уделить регулярному обновлению исходного кода ACME-клиентов.

Нами запланированы следующие изменения:

* корневой и промежуточный сертификаты, на основании которых мы выпускаем сертификаты
* алгоритм хэширования для подписи сертификатов
* типы ключей и проверок ключей, для подписи сертификатов
* и сам протокол ACME

Мы будем стараться предупреждать наших пользователей о грядущих изменениях заранее, насколько это возможно. Тем не менее, в случае обнаружения критической уязвимости, мы будем вынуждены внести изменения в течение короткого времени, или даже немедленно. Не стоит вносить в код промежуточные изменения протокола ACME, т.к. они часто обновляются. Рекомендуем использовать содержимое заголовка [`Link: rel="up"`](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-6.3.1) из ответа от серверов Let's Encrypt.

Аналогично, мы меняем URL ссылки на условия использования сервиса (terms of service, ToS) сразу после их обновления. Избегайте явного указания URL для ToS в исходном коде ACME-клиентов, вместо этого используйте содержимое заголовка [`Link: rel="terms-of-service"`](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-6.2) из ответа от серверов Let's Encrypt.

Вам также потребуется поддерживать в актуальном состоянии TLS-конфигурацию, для противодействия вновь найденным уязвимостям в наборах шифров или версиях TLS-протокола.

# Уведомления об изменениях

Для получения кратких уведомлений о важных изменениях (таких, как описано выше), подпишитесь на группу рассылки [API Announcements](https://community.letsencrypt.org/t/about-the-api-announcements-category/23836). Рекомендуется как разработчикам клиентского ПО, так и хостинг-провайдерам.

Для получения развёрнутой информации по обслуживанию и остановках работы сервиса, посетите нашу [страницу текущего состояния](https://letsencrypt.status.io/), и нажмите кнопку Subscribe справа вверху. Рекомендуется хостинг-провайдерам.

Убедитесь, что вы указали верный адрес электронной почты для своего ACME-аккаунта. Мы используем этот адрес для отправки уведомлений об истечении срока действия выпущенных для вас сертификатов, а также для взаимодействия с вами в случае проблем, специфичных для вашего аккаунта.

# Кто такой "Подписчик"

В нашем [Соглашении]({{< ref "/repository.md" >}}), под термином "Подписчик" мы понимаем любого владельца закрытого ключа для сертификата. Если вы - хостинг-провадер, то подписчиком являетесь вы, а не ваши клиенты. Если вы разрабатываете клиентское ПО, которое пользователю необходимо развернуть и настроить самостоятельно, то подписчиком будет тот, кто разворачивает клиентское ПО.

Адрес электронной почты, указываемый при создании аккаунта Let's Encrypt (он же "регистрация") должен принадлежать Подписчику. Мы используем этот адрес для отправки предупреждений об истечении срока действия сертификатов, и уведомлений об изменении нашей [политики конфиденциальности]({{< ref "/privacy.md" >}}). Если вы - хостинг-провайдер, эти уведомления должны приходить к вам, а не к вашим клиентам. Идеально было бы настроить список рассылки, чтобы несколько сотрудников могли получать от нас уведомления в ваше отстутствие.

Другими словами, если вы - хостинг-провайдер, вам не нужно передавать нам адреса электронной почты ваших клиентов, или ознакомлять их с политикой конфиденциальности. Вам достаточно выпустить сертификаты для доменов под вашим управлением, и тут же начать их использовать.

# Один аккаунт или несколько?

Согласно протоколу ACME, для авторизации и выпуска сертификатов возможно использование как одного общего аккаунта для нескольких клиентов, так и индивидуальных аккаунтов для каждого клиента в отдельности. Это может быть полезным, например, для хостинг-провайдеров. Создание индивидуальных аккаунтов для клиентов, с последующим их раздельным хранением, поможет в ситуации, когда один или несколько аккаунтов скомпрометированы, но остальные аккаунты в безопасности.

Тем не менее, для большинства хостинг-провайдеров мы рекомендуем использовать один общий аккаунт для всех клиентов, с хорошо охраняемым закрытым ключом для него. Это облегчит определение, кому именно принадлежит тот или иной сертификат, упростит актуализацию контактной информации и управление ограничениями, при необходимости. Мы не сможем эффективно корректировать ограничения в случае использования индивидуальных аккаунтов для ваших клиентов.

# Multi-domain (SAN) Certificates

Our [issuance policy]({{< ref "/docs/rate-limits.md" >}}) allows for up to 100 names per certificate. Whether you use a separate certificate for every hostname, or group together many hostnames on a small number of certificates, is up to you.

Using separate certificates per hostname means fewer moving parts are required to logically add and remove domains as they are provisioned and retired. Separate certificates also minimize certificate size, which can speed up HTTPS handshakes on low-bandwidth networks.

On the other hand, using large certificates with many hostnames allows you to manage fewer certificates overall. If you need to support older clients like Windows XP that do not support TLS Server Name Indication ([SNI](https://en.wikipedia.org/wiki/Server_Name_Indication)), you'll need a unique IP address for every certificate, so putting more names on each certificate reduces the number of IP addresses you'll need.

For most deployments both choices offer the same security.

# Storing and Reusing Certificates and Keys

A big part of Let's Encrypt's value is that it enables automatic issuance as part of provisioning a new website.  However, if you have infrastructure that may repeatedly create new frontends for the same website, those frontends should first try to use a certificate and private key from durable storage, and only issue a new one if no certificate is available, or all existing certificates are expired.

For Let's Encrypt, this helps us provide services efficiently to as many people as possible. For you, this ensures that you are able to deploy your website whenever you need to, regardless of the state of Let's Encrypt.

As an example, many sites are starting to use Docker to provision new frontend instances as needed. If you set up your Docker containers to issue when they start up, and you don't store your certificates and keys durably, you are likely to hit rate limits if you bring up too many instances at once. In the worst case, if you have to destroy and re-create all of your instances at once, you may wind up in a situation where none of your instances is able to get a certificate, and your site is broken for several days until the rate limit expires. This type of problem isn't unique to rate limits, though. If Let's Encrypt is unavailable for any reason when you need to bring up your frontends, you would have the same problem.

Note that some deployment philosophies state that crypto keys should never leave the physical machine on which they were generated. This model can work fine with Let's Encrypt, so long as you make sure that the machines and their data are long-lived, and you manage rate limits carefully.

# Picking a Challenge Type

If you're using the http-01 ACME challenge, you will need to provision the challenge response to each of your frontends before notifying Let's Encrypt that you're ready to fulfill the challenge. If you have a large number of frontends, this may be challenging. In that case, using the dns-01 challenge is likely to be easier. Of course, if you have many geographically distributed DNS responders, you have to make sure the TXT record is available on each responder.

Additionally, when using the dns-01 challenge, make sure to clean up old TXT records so the response to Let's Encrypt's query doesn't get too big.

If you want to use the http-01 challenge anyhow, you may want to take advantage of HTTP redirects. You can set up each of your frontends to redirect /.well-known/acme-validation/XYZ to validation-server.example.com/XYZ for all XYZ. This delegates responsibility for issuance to validation-server, so you should protect that server well.

# Central Validation Servers

Related to the above two points, it may make sense, if you have a lot of frontends, to use a smaller subset of servers to manage issuance. This makes it easier to use redirects for http-01 validation, and provides a place to store certificates and keys durably.

# Implement OCSP Stapling

Many browsers will fetch OCSP from Let's Encrypt when they load your site. This is a [performance and privacy problem](https://blog.cloudflare.com/ocsp-stapling-how-cloudflare-just-made-ssl-30/).  Ideally, connections to your site should not wait for a secondary connection to Let's Encrypt. Also, OCSP requests tell Let's Encrypt which sites people are visiting. We have a good privacy policy and do not record individually identifying details from OCSP requests, we'd rather not even receive the data in the first place. Additionally, we anticipate our bandwidth costs for serving OCSP every time a browser visits a Let's Encrypt site for the first time will be a big part of our infrastructure expense.

By turning on OCSP Stapling, you can improve the performance of your website, provide better privacy protections for your users, and help Let's Encrypt efficiently serve as many people as possible.

# Firewall Configuration

To use Let's Encrypt, you need to allow outbound port 443 traffic from the
machines running your ACME client. We don't publish the IP ranges for our
ACME service, and they will change without notice.

For the "http-01" ACME challenge, you need to allow inbound port 80 traffic.
We don't publish the IP ranges from which we perform validation, and they
will change without notice.

Note: We recommend always allowing plain HTTP access to your
web server, with a redirect to HTTPS. This provides a better user
experience than a web server that refuses or drops port 80 connections,
and provides the same level of security.

For all challenges, you need to allow inbound port 53 traffic
(TCP and UDP) to your authoritative DNS servers.

# Supported Key Algorithms

Let's Encrypt accepts RSA keys from 2048 to 4096 bits in length, and P-256 and P-384 ECDSA keys. That's true for both account keys and certificate keys. You can't reuse an account key as a certificate key.

Our recommendation is to serve a dual-cert config, offering an RSA certificate by default, and a (much smaller) ECDSA certificate to those clients that indicate support.

# HTTPS by default

For hosting providers, our recommendation is to automatically issue
certificates and configure HTTPS for all hostnames you control, and to offer a
user-configurable setting for whether to redirect HTTP URLs to their HTTPS
equivalents. We recommend that for existing accounts, the setting be disabled by
default, but for new accounts, the setting be enabled by default.

Reasoning: Existing websites are likely to include some HTTP subresources
(scripts, CSS, and images). If those sites are automatically redirected to
their HTTPS versions, browsers will block some of those subresources due to
Mixed Content Blocking. This can break functionality on the site. However,
someone who creates a new site and finds that it redirects to HTTPS will
most likely include only HTTPS subresources, because if they try to include
an HTTP subresource they will notice immediately that it doesn't work.

We recommend allowing customers to set an HTTP Strict-Transport-Security
(HSTS) header with a default max-age of sixty days. However, this setting
should be accompanied by a warning that if the customer needs to move to
a hosting provider that doesn't offer HTTPS, the cached HSTS setting in
browsers will make their site unavailable. Also, both customer and hosting
provider should be aware that the HSTS header will make certificate errors into
hard failures. For instance, while people can usually click through a browser
warning about a name mismatch or expired certificate, browsers do not allow such
a click through for hostnames with an active HSTS header.

# When to Renew

We recommend renewing certificates automatically when they have a third of their
total lifetime left. For Let's Encrypt's current 90-day certificates, that means
renewing 30 days before expiration.

If you are issuing for more than 10,000 hostnames, we also recommend automated
renewal in small runs, rather than batching up renewals into large chunks.
This reduces risk: If Let's Encrypt has an outage at the time you need to
renew, or there is a temporary failure in your renewal systems, it will only
affect a few of your certificates, rather than all of them. It also makes our
capacity planning easier.

You may want to bulk-issue certificates for all of your domains to get started
quickly, which is fine. You can then spread out renewal times by doing a
one-time process of renewing some certificates 1 day ahead of when you would
normally renew, some of them 2 days ahead, and so on.

If you offer client software that automatically configures a periodic batch
job, please make sure to run at a randomized hour and minute during the day,
rather than always running at a specific time. This ensures that Let's Encrypt
doesn't receive arbitrary spikes of traffic at the top of the hour. Since Let's
Encrypt needs to provision capacity to meet peak load, reducing traffic spikes
can help keep our costs down.

# Retrying failures

Renewal failure should not be treated as a fatal error. You should implement
graceful retry logic in your issuing services using an exponential backoff
pattern, maxing out at once per day per certificate. For instance, a reasonable
backoff schedule would be: 1st retry after one minute, 2nd retry after ten
minutes, third retry after 100 minutes, 4th and subsequent retries after one
day. You should of course have a way for administrators to
request early retries on a per-domain or global basis.

Backoffs on retry means that your issuance software should keep track of
failures as well as successes, and check if there was a recent failure before
attempting a fresh issuance. There's no point in attempting issuance hundreds
of times per hour, since repeated failures are likely to be persistent.

All errors should be sent to the administrator in charge, in order to see if
specific problems need fixing.
