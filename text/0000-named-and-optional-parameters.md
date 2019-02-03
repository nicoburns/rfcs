- Feature Name: named_and_optional_parameters
- Start Date: 2019-02-03
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Allows:
- Function parameters to be made optional by specifying a default value.
- Function parameters to be called by name in addition to position.

# Motivation
[motivation]: #motivation

Many high-level APIs involve the calling of functions or the creation of structs with a large amount of possible configuration values, but where in the usual case, only a few of these need to be specified with the rest falling back to a default value provided by the library.

An example of this is the [rusoto](https:://doc.rs/rusoto) crate, which includes several hundred functions which must be called as follows:

```rust

use rusoto_s3::{S3Client, S3, PutObjectRequest};
use std::default::Default;

fn main () {
    let client = S3Client(/* connection info here */);
    client.put_object(PutObjectRequest {
        key: "foo".into(),
        bucket: "bar".into(),
        content_length: Some(512),
        ..Default::default()
    });
}
```

It would be possible, using the builder pattern to write this function like this:

```rust
use rusoto_s3::{S3Client, S3};

fn main () {
    let client = S3Client(/* connection info here */);
    client.build_put_object()
        .key("foo")
        .bucket("bar")
        .content_length(512)
        .run()?;
}
```

However, this requires extra implementation effort by the library authors. And it also decreases type safety as the checking of the two required parameters now happens at runtime.

With the changes proposed in this RFC, this could be shortened to:


```rust

use rusoto_s3::{S3Client, S3};

fn main () {
    let client = S3Client(/* connection info here */);
    client.put_object(key: "foo", bucket: "bar", content_length: 512);
}
```

In addition to making the code more concise and readable, there are a number of benefits to this pattern:

- The parameters are directly in the function signature (and thus documentation will be better).
- The function signature can encode which parameters are required and which are optional, allowing for stricter static checking.
- As the parameters move from struct fields to function parameters, conversion traits like `Into` and `AsRef` may be used.
- Only the base struct (and trait in this case) needs to be imported with `use`. With the existing solution, an additional struct must be imported for each method used.


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

<details><p>

```rust
pub fn put_object(&self, input: PutObjectRequest) -> RusotoFuture<PutObjectOutput, PutObjectError>;

pub struct PutObjectRequest {
    /// <p>The canned ACL to apply to the object.</p>
    pub acl: Option<String>,
    /// <p>Object data.</p>
    pub body: Option<StreamingBody>,
    /// <p>Name of the bucket to which the PUT operation was initiated.</p>
    pub bucket: String,
    /// <p>Specifies caching behavior along the request/reply chain.</p>
    pub cache_control: Option<String>,
    /// <p>Specifies presentational information for the object.</p>
    pub content_disposition: Option<String>,
    /// <p>Specifies what content encodings have been applied to the object and thus what decoding mechanisms must be applied to obtain the media-type referenced by the Content-Type header field.</p>
    pub content_encoding: Option<String>,
    /// <p>The language the content is in.</p>
    pub content_language: Option<String>,
    /// <p>Size of the body in bytes. This parameter is useful when the size of the body cannot be determined automatically.</p>
    pub content_length: Option<i64>,
    /// <p>The base64-encoded 128-bit MD5 digest of the part data.</p>
    pub content_md5: Option<String>,
    /// <p>A standard MIME type describing the format of the object data.</p>
    pub content_type: Option<String>,
    /// <p>The date and time at which the object is no longer cacheable.</p>
    pub expires: Option<String>,
    /// <p>Gives the grantee READ, READ_ACP, and WRITE_ACP permissions on the object.</p>
    pub grant_full_control: Option<String>,
    /// <p>Allows grantee to read the object data and its metadata.</p>
    pub grant_read: Option<String>,
    /// <p>Allows grantee to read the object ACL.</p>
    pub grant_read_acp: Option<String>,
    /// <p>Allows grantee to write the ACL for the applicable object.</p>
    pub grant_write_acp: Option<String>,
    /// <p>Object key for which the PUT operation was initiated.</p>
    pub key: String,
    /// <p>A map of metadata to store with the object in S3.</p>
    pub metadata: Option<::std::collections::HashMap<String, String>>,
    pub request_payer: Option<String>,
    /// <p>Specifies the algorithm to use to when encrypting the object (e.g., AES256).</p>
    pub sse_customer_algorithm: Option<String>,
    /// <p>Specifies the customer-provided encryption key for Amazon S3 to use in encrypting data. This value is used to store the object and then it is discarded; Amazon does not store the encryption key. The key must be appropriate for use with the algorithm specified in the x-amz-server-side​-encryption​-customer-algorithm header.</p>
    pub sse_customer_key: Option<String>,
    /// <p>Specifies the 128-bit MD5 digest of the encryption key according to RFC 1321. Amazon S3 uses this header for a message integrity check to ensure the encryption key was transmitted without error.</p>
    pub sse_customer_key_md5: Option<String>,
    /// <p>Specifies the AWS KMS key ID to use for object encryption. All GET and PUT requests for an object protected by AWS KMS will fail if not made via SSL or using SigV4. Documentation on configuring any of the officially supported AWS SDKs and CLI can be found at http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingAWSSDK.html#specify-signature-version</p>
    pub ssekms_key_id: Option<String>,
    /// <p>The Server-side encryption algorithm used when storing this object in S3 (e.g., AES256, aws:kms).</p>
    pub server_side_encryption: Option<String>,
    /// <p>The type of storage to use for the object. Defaults to 'STANDARD'.</p>
    pub storage_class: Option<String>,
    /// <p>The tag-set for the object. The tag-set must be encoded as URL Query parameters</p>
    pub tagging: Option<String>,
    /// <p>If the bucket is configured as a website, redirects requests for this object to another object in the same bucket or to an external URL. Amazon S3 stores the value of this header in the object metadata.</p>
    pub website_redirect_location: Option<String>,
}
```

</p></details>

<details><p>

```rust
pub fn put_object(
    &self,

    /// <p>The canned ACL to apply to the object.</p>
    acl: impl Into<Option<String>>,
    /// <p>Object data.</p>
    body: impl Into<Option<StreamingBody>>,
    /// <p>Name of the bucket to which the PUT operation was initiated.</p>
    bucket: impl Into<String>,
    /// <p>Specifies caching behavior along the request/reply chain.</p>
    cache_control: impl Into<Option<String>>,
    /// <p>Specifies presentational information for the object.</p>
    content_disposition: impl Into<Option<String>>,
    /// <p>Specifies what content encodings have been applied to the object and thus what decoding mechanisms must be applied to obtain the media-type referenced by the Content-Type header field.</p>
    content_encoding: impl Into<Option<String>>,
    /// <p>The language the content is in.</p>
    content_language: impl Into<Option<String>>,
    /// <p>Size of the body in bytes. This parameter is useful when the size of the body cannot be determined automatically.</p>
    content_length: impl Into<Option<i64>>,
    /// <p>The base64-encoded 128-bit MD5 digest of the part data.</p>
    content_md5: impl Into<Option<String>>,
    /// <p>A standard MIME type describing the format of the object data.</p>
    content_type: impl Into<Option<String>>,
    /// <p>The date and time at which the object is no longer cacheable.</p>
    expires: impl Into<Option<String>>,
    /// <p>Gives the grantee READ, READ_ACP, and WRITE_ACP permissions on the object.</p>
    grant_full_control: impl Into<Option<String>>,
    /// <p>Allows grantee to read the object data and its metadata.</p>
    grant_read: impl Into<Option<String>>,
    /// <p>Allows grantee to read the object ACL.</p>
    grant_read_acp: impl Into<Option<String>>,
    /// <p>Allows grantee to write the ACL for the applicable object.</p>
    grant_write_acp: impl Into<Option<String>>,
    /// <p>Object key for which the PUT operation was initiated.</p>
    key: impl Into<String>,
    /// <p>A map of metadata to store with the object in S3.</p>
    metadata: impl Into<Option<::std::collections::HashMap<String, String>>>,
    request_payer: impl Into<Option<String>>,
    /// <p>Specifies the algorithm to use to when encrypting the object (e.g., AES256).</p>
    sse_customer_algorithm: impl Into<Option<String>>,
    /// <p>Specifies the customer-provided encryption key for Amazon S3 to use in encrypting data. This value is used to store the object and then it is discarded; Amazon does not store the encryption key. The key must be appropriate for use with the algorithm specified in the x-amz-server-side​-encryption​-customer-algorithm header.</p>
    sse_customer_key: impl Into<Option<String>>,
    /// <p>Specifies the 128-bit MD5 digest of the encryption key according to RFC 1321. Amazon S3 uses this header for a message integrity check to ensure the encryption key was transmitted without error.</p>
    sse_customer_key_md5: impl Into<Option<String>>,
    /// <p>Specifies the AWS KMS key ID to use for object encryption. All GET and PUT requests for an object protected by AWS KMS will fail if not made via SSL or using SigV4. Documentation on configuring any of the officially supported AWS SDKs and CLI can be found at http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingAWSSDK.html#specify-signature-version</p>
    ssekms_key_id: impl Into<Option<String>>,
    /// <p>The Server-side encryption algorithm used when storing this object in S3 (e.g., AES256, aws:kms).</p>
    server_side_encryption: impl Into<Option<String>>,
    /// <p>The type of storage to use for the object. Defaults to 'STANDARD'.</p>
    storage_class: impl Into<Option<String>>,
    /// <p>The tag-set for the object. The tag-set must be encoded as URL Query parameters</p>
    tagging: impl Into<Option<String>>,
    /// <p>If the bucket is configured as a website, redirects requests for this object to another object in the same bucket or to an external URL. Amazon S3 stores the value of this header in the object metadata.</p>
    website_redirect_location: impl Into<Option<String>>,

) -> RusotoFuture<PutObjectOutput, PutObjectError>;
```

</p></details>

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

# Prior art
[prior-art]: #prior-art

- [C# Named and Optional Arguments](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments)
- [Swift Functions](https://docs.swift.org/swift-book/LanguageGuide/Functions.html)
- https://hoverbear.org/2018/11/04/optional-arguments/

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

## Default field values for structs

Rust currently provides the `Default` trait, which allows a type to provide a default value where this makes sense. However, this is "all-or-nothing": it only works in cases where all fields of the struct have a sensible default. By extending the default parameter syntax to struct fields we can provide the same "some-mandatory-some-optional" semantics for structs as this RFC provides for function parameters:

```rust
struct Session {
    id:     u64
    user:   Option<User> = None,
    events: Vec<Event> = vec![],
}

fn main () {
    let session = Session {id: 123};
    assert_eq(session.user, Option::None);

    // Optionally we could require elipsis to explicitly opt in to default field values
    let session = Session {id: 123, ...};
    assert_eq(session.user, Option::None);

    let session = Session {user: User::new()}; // compile error: id field is required when creating Session structs
}
```

