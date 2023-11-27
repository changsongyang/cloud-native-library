---
weight: 2
title: "X.509 SPIFFE 可验证身份文档"
linkTitle: "X.509 SVID"
---

SPIFFE 标准提供了一种框架的规范，能够在异构环境和组织边界中引导和发放服务的身份。它定义了一种称为 SPIFFE 可验证身份文档（SVID）的身份文档。

SVID 本身并不代表一种新的文档类型。相反，我们提出了一个规范，定义了如何将 SVID 信息编码到现有文档类型中。

本文档定义了一种标准，其中将 X.509 证书用作 SVID。假设读者对 X.509 有基本的了解。关于 X.509 的具体信息，请参考[RFC 5280](https://tools.ietf.org/html/rfc5280)。

## 引言

SPIFFE 的最重要的功能之一是保护进程间通信。核心标准允许进行身份验证，但利用加密身份来构建安全的通信通道也是非常有益的。由于 TLS 被广泛采用，并且使用基于 X.509 的身份验证，将 X.509 用作 SPIFFE SVID 显然是有优势的。

本规范讨论了将 SVID 信息编码到 X.509 证书中的约束条件，以及如何验证 X.509 SVID。

## SPIFFE ID

在 X.509 SVID 中，对应的 SPIFFE ID 被设置为主题备用名称扩展（SAN 扩展，参见[RFC 5280 第 4.2.1.6 节](https://tools.ietf.org/html/rfc5280#section-4.2.1.6)）。一个 X.509 SVID 必须恰好包含一个 URI SAN，因此也只包含一个 SPIFFE ID。包含多个 SPIFFE ID 的 SVID 会引入与审计和授权逻辑相关的挑战，包含多个 URI SAN 的 SVID 会引入与 SPIFFE ID 验证相关的挑战。遇到包含多个 URI SAN 的 SVID 的验证器必须拒绝该 SVID。有关更多信息，请参见验证部分。

一个 X.509 SVID 可以包含任意数量的其他 SAN 字段类型，包括 DNS SAN。

## 层级关系

本节讨论了叶证书、根证书和中间证书之间的关系，以及对每个证书的要求。

### 叶证书

叶证书是用于标识调用方或资源的 SVID，适用于身份验证过程。叶证书（相对于签名证书，第 3.2 节）是唯一能够用于标识资源或调用方的类型。

叶证书的 SPIFFE ID 必须具有非根路径组件。如果省略了主题字段，则不需要主题字段，但如果省略了主题字段，则 URI SAN 扩展必须标记为关键扩展，根据[RFC 5280 第 4.1.2.6 节](https://tools.ietf.org/html/rfc5280)的规定。有关区分叶证书和签名证书的 X.509 特定属性的信息，请参见第 4.1 节。

### 签名证书

X.509 SVID 签名证书是具有在密钥用途扩展中设置`keyCertSign`的证书。它还在基本约束扩展中将`CA`标志设置为`true`（参见第 4.1 节）。也就是说，它是一个 CA 证书。

签名证书应该本身是一个 SVID。如果存在，签名证书的 SPIFFE ID 必须没有路径组件，并且可以位于其发行的任何叶 SVID 的信任域中。签名证书可以用于在相同或不同的信任域中发行进一步的签名证书。

签名证书不能用于身份验证目的。它们只作为验证材料，并且可以像[RFC 5280](https://tools.ietf.org/html/rfc5280)中描述的那样以典型的 X.509 方式链接在一起。请参见第 4.3 节和第 4.4 节以获取有关签名证书的 X.509 特定限制的更多信息。

## 约束和用途

叶证书和签名证书具有不同的 X.509 属性 - 一些用于安全目的，一些用于支持其特殊的功能。本节描述了两种类型 X.509 SVID 的约束和密钥用法配置。

### 基本约束

基本约束 X.509 扩展标识证书是否为签名证书，以及包括该证书在内的有效证书路径的最大深度。它在[RFC 5280 第 4.2.1.9 节](https://tools.ietf.org/html/rfc5280#section-4.2.1.9)中定义。

有效的 X.509 SVID 签名证书可以设置`pathLenConstraint`字段。签名证书必须将`cA`字段设置为`true`，而叶证书必须将`cA`字段设置为`false`。

### 名称约束

名称约束指示了一个命名空间，其中后续证书中的所有 SPIFFE ID 必须位于其中。它们用于将受损的签名证书的影响范围限制在命名的信任域内，并在[RFC 5280 第 4.2.1.10 节](https://tools.ietf.org/html/rfc5280#section-4.2.1.10)中定义。本节仅适用于签名证书。

名称约束的类型与主题备用名称相同。由于 SVID 关心的仅是 SPIFFE ID，并且 SPIFFE ID 被定义为 SAN 类型 URI，因此我们只定义 URI 类型名称约束的语义。

目前，对 URI 类型名称约束的支持相对较少。不支持它们的库将拒绝此类证书，阻止路径验证成功。虽然名称约束是 SPIFFE 希望使用的 X.509 功能，但作者认识到广泛支持的缺乏可能会给实现和/或部署带来重大痛苦。因此，X.509 SVID 签名证书可以根据实现者的意愿应用 URI 名称约束，但在此领域应谨慎使用。SPIFFE 社区正在努力在各种平台上启用对 URI 名称约束的支持，并且应该预期在未来的版本中，随着广泛支持的实现，本节中定义的要求将变得更加严格。

### 密钥用法

密钥用法扩展定义了证书中包含的密钥的用途。当要限制可以用于多个操作的密钥时，可以使用此用法限制。密钥用法扩展在[RFC 5280 第 4.2.1.3 节](https://tools.ietf.org/html/rfc5280#section-4.2.1.3)中定义。

密钥用法扩展必须在所有 SVID 上设置，并且必须标记为关键扩展。

SVID 签名证书必须设置`keyCertSign`。它们可以设置`cRLSign`。

叶 SVID 必须设置`digitalSignature`。它们可以设置`keyEncipherment`和/或`keyAgreement`；这些对于 RSA 密钥的证书来说通常只有在需要的情况下才有意义，即使在那种情况下通常也不需要。叶 SVID 不能设置`keyCertSign`或`cRLSign`。

### 扩展密钥用途

该扩展指示证书中包含的密钥可以用于的一个或多个目的，除了或代替密钥用途扩展中指示的基本目的之外。它在[RFC 5280，第 4.2.1.2 节](https://tools.ietf.org/html/rfc5280#section-4.2.1.2)中定义。

Leaf SVID 应包括此扩展，并且可以将其标记为关键。当包含时，字段`id-kp-serverAuth`和`id-kp-clientAuth`必须设置。

签名证书可以包括扩展密钥用途。请注意，X.509 证书验证库中间 CA 证书中扩展密钥用途的处理方式因实现而异。有些 X.509 实现会对信任链中其下的所有证书都施加中间 CA 证书中的扩展密钥用途约束，而其他实现则不会。

## 验证

本节描述了如何验证 X.509 SVID。该过程使用标准的 X.509 验证，以及一系列 SPIFFE 特定的验证步骤。

### 路径验证

对给定 SVID 的信任验证基于标准的 X.509 路径验证，并且必须遵循[RFC 5280](https://tools.ietf.org/html/rfc5280)的路径验证语义。

证书路径验证需要提供叶子 SVID 证书和一个或多个 SVID 签名证书。用于验证的签名证书集合称为 CA 捆绑包。实体检索相关 CA 捆绑包的机制不在本文档范围内，而是在 SPIFFE 工作负载 API 规范中定义。

### 叶子验证

在验证资源或调用方时，需要进行超出 X.509 标准范围的验证。即，我们必须确保 1）证书是叶子证书，2）签发机构有权签发它。

在验证用于身份验证目的的 X.509 SVID 时，验证器必须确保基本约束扩展中的`CA`字段设置为`false`，并且密钥用途扩展中未设置`keyCertSign`和`cRLSign`。验证器还必须确保 SPIFFE ID 的方案设置为`spiffe://`。包含多个 URI SAN 的 SVID 必须被拒绝。

随着 URI 名称约束的支持越来越广泛，本文档的未来版本可能会更新本节中设定的要求，以便更好地利用名称约束验证。

## 在 SPIFFE 捆绑包中的表示

本节描述了如何将 X509-SVID CA 证书发布到 SPIFFE 捆绑包中，并从中使用。有关 SPIFFE 捆绑包的更多信息，请参阅 SPIFFE 信任域和捆绑包规范。

### 发布 SPIFFE 捆绑包元素

给定信任域的 X509-SVID CA 证书在 SPIFFE 捆绑包中表示为[RFC 7517 兼容](https://tools.ietf.org/html/rfc7517)的 JWK 条目，每个 CA 证书一个条目。

每个 JWK 条目的`use`参数必须设置为`x509-svid`。另外，每个 JWK 条目的`kid`参数不能设置。

除了[RFC 7517](https://tools.ietf.org/html/rfc7517)要求的参数之外，表示 X509-SVID CA 证书的每个条目必须包含具有与该条目表示的基于 Base64 编码的 DER CA 证书相等的值的`x5c`参数。该值必须包含且仅包含一个 CA 证书，并且该证书应为自签名。

### 使用 SPIFFE 捆绑包

从外部信任域使用 SPIFFE 捆绑包时，需要提取 X509-SVID CA 证书以供实际使用。SPIFFE 捆绑包可以包含许多不同类型的 SVID 条目，因此第一步是识别表示 X509-SVID CA 证书的条目。

对于捆绑包中`use`参数设置为`x509-svid`的每个 JWK 条目，请检查`x5c`参数是否设置并且至少具有一个值。如果`x5c`未设置或为空，则必须忽略该条目。

`x5c`参数的第一个值是该条目表示的 Base64 DER 编码的 CA 证书。如果`x5c`参数包含多个值，则除第一个值外的所有值都必须被忽略。然后，X509-SVID CA 捆绑包是从`x509-svid` JWK 条目中提取的 CA 证书的并集。如果捆绑包中不存在`x509-svid` JWK 条目，则信任域不支持 X509-SVID。

## 结论

本文档提出了 X.509 基于 SPIFFE 可验证身份文档的约定和标准。它构成了现实世界 SPIFFE 服务身份验证和 SVID 验证的基础。通过遵守 X.509 SVID 标准，可以构建一个可互操作且与平台无关的身份和身份验证系统。

## 附录 A. X.509 字段参考

| 扩展         | 字段                      | 描述                                                         |
| ------------ | ------------------------- | ------------------------------------------------------------ |
| 主题备用名称 | uniformResourceIdentifier | 此字段设置为 SPIFFE ID。仅允许一个此字段的实例。              |
| 基本约束     | CA                        | 如果 SVID 是签名证书，则必须将此字段设置为 true。               |
| 基本约束     | pathLenConstraint         | 如果实现者希望执行有限的 CA 层次结构深度约束（例如，从现有私钥基础设施（PKI）继承），则可以设置此字段。 |
| 名称约束     | permittedSubtrees         | 如果实现者希望使用 URI 名称约束，则可以设置此字段。这将在本文档的未来版本中要求。 |
| 密钥用途     | keyCertSign               | 如果 SVID 是签名证书，则必须设置此字段。                       |
| 密钥用途     | cRLSign                   | 如果 SVID 是签名证书，则可以设置此字段。                       |
| 密钥用途     | keyAgreement              | 如果 SVID 是叶子证书，则可以设置此字段。                       |
| 密钥用途     | keyEncipherment           | 如果 SVID 是叶子证书，则可以设置此字段。                       |
| 密钥用途     | digitalSignature          | 如果 SVID 是叶子证书，则必须设置此字段。                       |
| 扩展密钥用途 | id-kp-serverAuth          | 此字段可为叶子证书或签名证书设置。                           |
| 扩展密钥用途 | id-kp-clientAuth          | 此字段可为叶子证书或签名证书设置。                           |