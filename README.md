# flat-selection

## Usage

```typescript
import { createFlatten, type FlatSelection } from 'flat-selection';
import { CommentThread, User, Comment } from '@-/database';

interface State {
  $collections: {
    CommentThread: Record<string, FlatSelection<CommentThread>>;
    Comment: Record<string, FlatSelection<Comment>>;
    User: Record<string, FlatSelection<User>>;
  },
  // other app state...
}

const state: State = {
  $collections: {
    CommentThread: {},
    Comment: {},
    User: {}
  },
  // other app state...
}

const flatten = createFlatten(state.$collections);

const nestedCommentThread = {
  _id: 'thread0',
  author: {
    _id: 'user0',
    name: 'User Name'
  },
  comments: [
    {
      _id: 'comment0',
      rawText: 'A comment',
      files: [
        {
          _id: 'file0',
          url: 'url'
        }
      ]
    }
  ]
};

const flatCommentThread = flatten(nestedCommentThread, 'CommentThread', {
  author: (user) => flatten(user, 'User', {}),
  comments: (comment) => flatten(comment, 'Comment', {
    files: (file) => flatten(file, 'File', {})
  })
})

expect(flatCommentThread).toEqual({
  CommentThread: {
    thread0: {
      _id: 'thread0',
      authorProjectLivePreviewMemberRef: {
        id: 'author0'
      },
    }
  },
  Comment: {
    comment0: {
      _id: 'comment0',
      rawText: 'A comment',
      filesRefs: [{ id: 'file0' }],
    }
  },
  File: {
    file0: {
      _id: 'file0',
      url: 'url'
    }
  },
  User: {
    user: {
      _id: 'author0',
      name: 'User Name'
    }
  }
})
```
