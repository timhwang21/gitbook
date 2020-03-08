# RSpec

Certain RSpec patterns.

## Subject should match what is being described

Instead of:

```ruby
subject { instance }

before do
  instance.update!(something: something)
end

expect(subject).to ...
```

It is clearer to handle the update in `subject` and then frame the assertion as a change.

```ruby
subject { instance.update!(something: something) }

expect { subject }.to change { ... }
```

## `.to match` vs. `.to eq`

Unlike in Jest, subset matchers only work with the looser `.to match`. When using Jest, `expect.objectContaining`, etc. can be used with both `.toEqual` and `.toBe`. However, `a_hash_including`, etc. will simply fail when used with `.to eq`! This is especially frustrating because nothing about the error message indicates that the failure comes from using the wrong matcher.

## Diffing JSON blobs

RSpec inlines JSON blobs by default, which makes diffs very hard to detect. The [`super_diff`](https://github.com/mcmire/super_diff) gem helps with this by printing the diff line-by-line.
