## GraphQL ##

1. Schema 定义

		type Post {
		  title: String!
		  author: Person!
		}
		
		type Person {
		  name: String!
		  age: Int!
		  posts: [Post!]!
		}

	> 以上是一对多的关系

1. 基础查询

    
	![](https://i.imgur.com/s58SFrf.png)

	![](https://i.imgur.com/b3ngt16.png)
	
	以下是带条件
    
	
	- 单条件
	![](https://i.imgur.com/SCRVcMu.png)
	>查询最后两条数据

	
 	
	- 多条件
    ![](https://i.imgur.com/YU9krbY.png)

    
1. 变更（mutations）

	- 新增数据

		![](https://i.imgur.com/mGfzP5i.png)

	- 更新数据update
	
			mutation {
			  updatePerson(
			   id:"cja9dpyt6dv0k018064yufoz0"
			   age: 99
			  ) {
			    id
			  }
			}

	- 删除数据delete

			mutation {
			  deletePerson(
			   id:"cja9dpyt6dv0k018064yufoz0"
			  ) {
			    id
			  }
			}

	**进阶加深理解：**

	以下是两个类型的mutation:

		input IntroduceShipInput {
		  factionId: ID!
		  shipName: String!
		  clientMutationId: String!
		}
		
		type IntroduceShipPayload {
		  faction: Faction
		  ship: Ship
		  clientMutationId: String!
		}

	继续：

		mutation AddBWingQuery($input: IntroduceShipInput!) {
		  introduceShip(input: $input) {
		    ship {
		      id
		      name
		    }
		    faction {
		      name
		    }
		    clientMutationId
		  }
		}

	传入以下参数：

		{
		  "input": {
		    "shipName": "B-Wing",
		    "factionId": "1",
		    "clientMutationId": "abcde"
		  }
		}

	得到的结果：
		
		{
		  "introduceShip": {
		    "ship": {
		      "id": "U2hpcDo5",
		      "name": "B-Wing"
		    },
		    "faction": {
		      "name": "Alliance to Restore the Republic"
		    },
		    "clientMutationId": "abcde"
		  }
		}



1. 订阅（subscriptions）

		subscription{
		  Person{
		     node {
			    id
			 }
		  }
		}
	>起到监控的作用，当创建一个新的Person的时候，服务器就会发送信息给客户端

	

	- 选择性监听
	

			subscription{
			  Post(filter:{mutation_in:[CREATED]}){
			     node {
				    id
				    description
				    imageUrl
				 }
			  }
			}
		>以上只监听新增的数据，mutation_in可以是 CREATED, UPDATED or DELETED



1. Connection  看以下exmaple:
  
	查第一条：

		query RebelsShipsQuery {
		  rebels {
		    name,
		    ships(first: 1) {
		      edges {
		        node {
		          name
		        }
		      }
		    }
		  }
		}

	转换后：

		{
		  "rebels": {
		    "name": "Alliance to Restore the Republic",
		    "ships": {
		      "edges": [
		        {
		          "node": {
		            "name": "X-Wing"
		          }
		        }
		      ]
		    }
		  }
		}

	继续查前两条：
		
		query MoreRebelShipsQuery {
		  rebels {
		    name,
		    ships(first: 2) {
		      edges {
		        cursor
		        node {
		          name
		        }
		      }
		    }
		  }
		}

	转换后：

		{
		  "rebels": {
		    "name": "Alliance to Restore the Republic",
		    "ships": {
		      "edges": [
		        {
		          "cursor": "YXJyYXljb25uZWN0aW9uOjA=",
		          "node": {
		            "name": "X-Wing"
		          }
		        },
		        {
		          "cursor": "YXJyYXljb25uZWN0aW9uOjE=",
		          "node": {
		            "name": "Y-Wing"
		          }
		        }
		      ]
		    }
		  }
		}

  
    查`YXJyYXljb25uZWN0aW9uOjE=`这条记录的后三条：
		
		query EndOfRebelShipsQuery {
		  rebels {
		    name,
		    ships(first: 3 after: "YXJyYXljb25uZWN0aW9uOjE=") {
		      edges {
		        cursor,
		        node {
		          name
		        }
		      }
		    }
		  }
		}

	转换后：

		{
		  "rebels": {
		    "name": "Alliance to Restore the Republic",
		    "ships": {
		      "edges": [
		        {
		          "cursor": "YXJyYXljb25uZWN0aW9uOjI=",
		          "node": {
		            "name": "A-Wing"
		          }
		        },
		        {
		          "cursor": "YXJyYXljb25uZWN0aW9uOjM=",
		          "node": {
		            "name": "Millenium Falcon"
		          }
		        },
		        {
		          "cursor": "YXJyYXljb25uZWN0aW9uOjQ=",
		          "node": {
		            "name": "Home One"
		          }
		        }
		      ]
		    }
		  }
		}

	继续：
		
		query RebelsQuery {
		  rebels {
		    name,
		    ships(first: 4 after: "YXJyYXljb25uZWN0aW9uOjQ=") {
		      edges {
		        cursor,
		        node {
		          name
		        }
		      }
		    }
		  }
		}

	转换后发现没有数据了：
		
		{
		  "rebels": {
		    "name": "Alliance to Restore the Republic",
		    "ships": {
		      "edges": []
		    }
		  }
		}

	**重点在这，看如下：**

		query EndOfRebelShipsQuery {
		  rebels {
		    name,
		    originalShips: ships(first: 2) {
		      edges {
		        node {
		          name
		        }
		      }
		      pageInfo {
		        hasNextPage
		      }
		    }
		    moreShips: ships(first: 3 after: "YXJyYXljb25uZWN0aW9uOjE=") {
		      edges {
		        node {
		          name
		        }
		      }
		      pageInfo {
		        hasNextPage
		      }
		    }
		  }
		}

	转换后：

		{
		  "rebels": {
		    "name": "Alliance to Restore the Republic",
		    "originalShips": {
		      "edges": [
		        {
		          "node": {
		            "name": "X-Wing"
		          }
		        },
		        {
		          "node": {
		            "name": "Y-Wing"
		          }
		        }
		      ],
		      "pageInfo": {
		        "hasNextPage": true
		      }
		    },
		    "moreShips": {
		      "edges": [
		        {
		          "node": {
		            "name": "A-Wing"
		          }
		        },
		        {
		          "node": {
		            "name": "Millenium Falcon"
		          }
		        },
		        {
		          "node": {
		            "name": "Home One"
		          }
		        }
		      ],
		      "pageInfo": {
		        "hasNextPage": false
		      }
		    }
		  }
		}

	>总结：可以使用`pageInfo`，读取到是否有数据 





##Relay Modern

1. Environment（搭建环境配置）和 Network Layer（网络层）

	- 环境定义所需配置，将缓存存储和网络处理功能捆绑在一起，以便操作。

	- 	网络层提供一个接口对象，环境使用这个网络层来执行查询、更新和订阅的工作。

			const {Environment, Network} = require('relay-runtime');
			
			// Define a function that fetches the results of an operation (query/mutation/etc)
			// and returns its results as a Promise:
			function fetchQuery(
			  operation,
			  variables,
			  cacheConfig,
			  uploadables,
			) {
			  return fetch('/graphql', {
			    method: 'POST',
			    headers: {
			      // Add authentication and other headers here
			      'content-type': 'application/json'
			    },
			    body: JSON.stringify({
			      query: operation.text, // GraphQL text from input
			      variables,
			    }),
			  }).then(response => {
			    return response.json();
			  });
			}
			
			// Create a network layer from the fetch function
			const network = Network.create(fetchQuery);
			
			// Create an environment using this network:
			const environment = new Environment({
			  ..., // other options
			  network,
			});
		>以上是简单的基本配置，可以自行拓展，例如当cacheConfig.force为false时，上传更新的表单数据等。

1. QueryRenderer

	**它是Relay树的根，接受查询，获取数据并render用数据调用回调，同时它也是React的一个组件，渲染到React组件的任何地方，而不仅仅是在顶层，可以在其他Relay组件中呈现。例如延迟获取数据，在QueryRenderer挂载之前，将不会开始加载数据，所以QueryRenderer如果不必要地使用嵌套组件，可能会导致不可避免的瀑布请求。**
	
		const {
		  QueryRenderer,
		  graphql,
		} = require('react-relay'); // or require('react-relay/compat') for compatibility
		
		// Render this somewhere with React:
		<QueryRenderer
		  environment={environment}
		  query={graphql`
		    query ExampleQuery($pageID: ID!) {
		      page(id: $pageID) {
		        name
		      }
		    }
		  `}
		  variables={{
		    pageID: '110798995619330',
		  }}
		  render={({error, props}) => {
		    if (error) {
		      return <div>{error.message}</div>;
		    } else if (props) {
		      return <div>{props.page.name} is great!</div>;
		    }
		    return <div>Loading</div>;
		  }}
		/>

	>如以上的根文件，一般写在App.js中
	>
	>命名的规定：文件名+操作类型，以上为查询功能，该文件为Exampl.js，故命名为ExampleQuery。

1. FragmentContainer

 	**声明数据需求的主要方法是通过创建一个createFragmentContainer的高阶组件，让react组件编码他们的数据需求，事先声明需要渲染的数据，relay保证了组件在渲染之前数据是可用的。**
	
	- 基础的React组件
	
		 	class TodoItem extends React.Component {
				  render() {
				    // Expects the `item` prop to have the following shape:
				    // {
				    //   item: {
				    //     text,
				    //     isComplete
				    //   }
				    // }
				    const item = this.props.item;
				    return (
				      <View>
				        <Checkbox checked={item.isComplete} />
				        <Text>{item.text}</Text>
				      </View>
				    );
				  }
				}

	- 数据依赖GraphQL

			graphql`
			  # This fragment only applies to objects of type 'Todo'.
			  fragment TodoItem_item on Todo {
			    text
			    isComplete
			  }
			`
		>片段的声明规则：文件名+propName
		
	- Relay Container
	
		**给了React组件和GraphQL的片段之后，现在我们能定义一个Container去告诉Relay关于组件的数据请求。**

			class TodoItem extends React.Component {/* as above */}
			
			// Export a *new* React component that wraps the original `<TodoItem>`.
			module.exports = createFragmentContainer(TodoItem, {
			  // For each of the props that depend on server data, we define a corresponding
			  // key in this object. Here, the component expects server data to populate the
			  // `item` prop, so we'll specify the fragment from above at the `item` key.
			  item: graphql`
			    fragment TodoItem_item on Todo {
			      text
			      isComplete
			    }
			  `,
			});

		>以上是传统的写法，下面的例子等同于上面的例子。

			module.exports = createFragmentContainer(
			  TodoItem,
			  graphql`
			    fragment TodoItem_item on Todo {
			      text
			      isComplete
			    }
			  `,
			);

		>注意：这里的片段名称如何没有使用_<propName>,则要使用data
		
			class TodoItem extends React.Component {
			  render() {
			    const item = this.props.data;
			
			  }
			}
			module.exports = createFragmentContainer(
			  TodoItem,
			  graphql`
			    fragment TodoItem on Todo {
			      text
			      isComplete
			    }
			  `,
			);

	- Relay Container Composition
	
		**下面是TodoList的组件:**

			 class TodoList extends React.Component {
				  render() {
				    // Expects a `list` with a string `title`, as well as the information
				    // for the `<TodoItem>`s (we'll get that next).
				    const list = this.props.list;
				    return (
				      <View>
				        {/* It works just like a React component, because it is one! */}
				        <Text>{list.title}</Text>
				        {list.todoItems.map(item => <TodoItem item={item} />)}
				      </View>
				    );
				  }
				}

		**下面是TodoList复杂的片段容器：**

			class TodoList extends React.Component {/* as above */}
				module.exports = createFragmentContainer(
				  TodoList,
				  // This `_list` fragment name suffix corresponds to the prop named `list` that
				  // is expected to be populated with server data by the `<TodoList>` component.
				  graphql`
				    fragment TodoList_list on TodoList {
				      # Specify any fields required by '<TodoList>' itself.
				      title
				      # Include a reference to the fragment from the child component.
				      todoItems {
				        ...TodoItem_item
				      }
				    }
				  `,
				);

	- 调用组件实例的方法

		**TodoListView 组件为TodoInput 组件的父组件，若TodoListView想调用TodoInput组件内的方法，要使用componentRef 的ref,如下：**
 
			//TodoInput组件
			module.exports = createFragmentContainer(
			  class TodoInput extends React.Component {
			    focus() {
			      this.input.focus();
			    }
			
			    render() {
			      return <input
			        ref={node => { this.input = node; }}
			        placeholder={this.props.data.suggestedNextTitle}
			      />;
			    }
			  },
			  graphql`
			    fragment TodoInput on TodoList {
			      suggestedNextTitle
			    }
			  `,
			);

			//TodoListView组件
			module.exports = createFragmentContainer(
			  class TodoListView extends React.Component {
			    render() {
			      return <div onClick={() => this.input.focus()}>
			        <TodoInput
			          data={this.props.data}
			          componentRef={ref => { this.input = ref; }}
			        />
			      </div>;
			    }
			  },
			  graphql`
			    fragment TodoListView on TodoList {
			      ...TodoInput
			    }
			  `,
			);


1. RefetchContainer

	**RefetchContainer渲染方式跟常规的FragmentContainer相同，不同的是，RefetchContainer可以选择不同的变量执行新查询，并在请求返回时呈现查询的响应。
	下面是API：**

		type Variables = {[name: string]: any};
		type RefetchOptions = {
		  force?: boolean, // Refetch even if already fetched this query and variables.
		};
		type Disposable = {
		  dispose(): void,
		};
		
		/**
		 * Execute the refetch query
		 */
		refetch: (
		  refetchVariables: Variables | (fragmentVariables: Variables) => Variables,
		  renderVariables: ?Variables,
		  callback: ?(error: ?Error) => void,
		  options?: RefetchOptions,
		) => Disposable,

	- refetchVariables： 一个大的值的集合或者是一个函数，获取之前片段的值和返回新的值
	
	- renderVariables :可选字段，告诉Relay在获取组件后重新呈现组件时使用哪些变量，没有这个，refetchVariables将被使用。你可以使用这个更高级的用法，例如，实现分页，你可以获取一个额外的页面，像变量{first: 5, after: '...'}，但是你可以用完整的集合来渲染{first: 10}。
	
	- 返回一个Disposable ，你可以调用dispose()来取消refetch

	 **Example如下：**
		
		const {
		  createRefetchContainer,
		  graphql,
		} = require('react-relay');
		
		class FeedStories extends React.Component {
		  render() {
		    return (
		      <div>
		        {this.props.feed.stories.edges.map(
		          edge => <Story story={edge.node} key={edge.node.id} />
		        )}
		        <button
		          onPress={this._loadMore}
		          title="Load More"
		        />
		      </div>
		    );
		  }
		
		  _loadMore() {
		    // Increments the number of stories being rendered by 10.
		    const refetchVariables = fragmentVariables => ({
		      count: fragmentVariables.count + 10,
		    });
		    this.props.relay.refetch(refetchVariables, null);
		  }
		}
		
		module.exports = createRefetchContainer(
		  FeedStories,
		  {
		    feed: graphql`
		      fragment FeedStories_feed on Feed
		      @argumentDefinitions(
		        count: {type: "Int", defaultValue: 10}
		      ) {
		        stories(first: $count) {
		          edges {
		            node {
		              id
		              ...Story_story
		            }
		          }
		        }
		      }
		    `
		  },
		  graphql`
		    query FeedStoriesRefetchQuery($count: Int) {
		      feed {
		        ...FeedStories_feed @arguments(count: $count)
		      }
		    }
		  `,
		);

1. PaginationContainer

	**顾名思义这个为分页容器,下面是API:**

		type Variables = {[name: string]: any};
		type RefetchOptions = {
		  force?: boolean, // Refetch from the server ignoring anything in the cache.
		};
		type Disposable = {
		  dispose(): void,
		};
		
		/**
		 * Check if there is at least one more page.
		 */
		hasMore: () => boolean,
		
		/**
		 * Check if there are pending requests.
		 */
		isLoading: () => boolean,
		
		/**
		 * Execute the pagination query. Relay will infer the pagination direction (either 'forward'
		 * or 'backward') from the query parameters. `pageSize` is the additional number of items
		 * to load.
		 */
		loadMore: (
		  pageSize: number,
		  callback: ?(error: ?Error) => void,
		  options: ?RefetchOptions
		) => ?Disposable,
		
		/**
		 * Refetch the items in the connection (with potentially new variables).
		 */
		refetchConnection:(
		  totalCount: number,
		  callback: (error: ?Error) => void,
		  refetchVariables: ?Variables,
		) => ?Disposable,


	**Example如下：**

		const {
		  createPaginationContainer,
		  graphql,
		} = require('react-relay');
		
		class Feed extends React.Component {
		  render() {
		    return (
		      <div>
		        {this.props.user.feed.edges.map(
		          edge => <Story story={edge.node} key={edge.node.id} />
		        )}
		        <button
		          onPress={() => this._loadMore()}
		          title="Load More"
		        />
		      </div>
		    );
		  }
		
		  _loadMore() {
		    if (!this.props.relay.hasMore() || this.props.relay.isLoading()) {
		      return;
		    }
		
		    this.props.relay.loadMore(
		      10, // Fetch the next 10 feed items
		      e => {
		        console.log(e);
		      },
		    );
		  }
		}
		
		module.exports = createPaginationContainer(
		  Feed,
		  {
		    user: graphql`
		      fragment Feed_user on User {
		        feed(
		          first: $count
		          after: $cursor
		          orderby: $orderBy # other variables
		        ) @connection(key: "Feed_feed") {
		          edges {
		            node {
		              id
		              ...Story_story
		            }
		          }
		        }
		      }
		    `,
		  },
		  {
		    direction: 'forward',
		    getConnectionFromProps(props) {
		      return props.user && props.user.feed;
		    },
		    getFragmentVariables(prevVars, totalCount) {
		      return {
		        ...prevVars,
		        count: totalCount,
		      };
		    },
		    getVariables(props, {count, cursor}, fragmentVariables) {
		      return {
		        count,
		        cursor,
		        // in most cases, for variables other than connection filters like
		        // `first`, `after`, etc. you may want to use the previous values.
		        orderBy: fragmentVariables.orderBy,
		      };
		    },
		    query: graphql`
		      query FeedPaginationQuery(
		        $count: Int!
		        $cursor: String
		        $orderby: String!
		      ) {
		        user {
		          # You could reference the fragment defined previously.
		          ...Feed_user
		        }
		      }
		    `
		  }
		);


1. Mutation

	**API如下：**

		const {commitMutation} = require('react-relay');
		
		type Variables = {[name: string]: any};
		
		commitMutation(
		  environment: Environment,
		  config: {
		    mutation: GraphQLTaggedNode,
		    variables: Variables,
		    onCompleted?: ?(response: ?Object, errors: ?[Error]) => void,
		    onError?: ?(error: Error) => void,
		    optimisticResponse?: Object,
		    optimisticUpdater?: ?(store: RecordSourceSelectorProxy) => void,
		    updater?: ?(store: RecordSourceSelectorProxy, data: SelectorData) => void,
		    configs?: Array<RelayMutationConfig>,
		  },
		);

	**Example如下，以下只用到mutation 和	variables**

		const {
		  commitMutation,
		  graphql,
		} = require('react-relay');
		
		const mutation = graphql`
		  mutation MarkReadNotificationMutation(
		    $input: MarkReadNotificationData!
		  ) {
		    markReadNotification(data: $input) {
		      notification {
		        seenState
		      }
		    }
		  }
		`;
		
		function markNotificationAsRead(environment, source, storyID) {
		  const variables = {
		    input: {
		      source,
		      storyID,
		    },
		  };
		
		  commitMutation(
		    environment,
		    {
		      mutation,
		      variables,
		      onCompleted: (response, errors) => {
		        console.log('Response received from server.')
		      },
		      onError: err => console.error(err),
		    },
		  );
		}

	- 配置optimisticResponse，达到在服务器的响应恢复之前，客户端立即更新以反映预期的新值
		
			const optimisticResponse =  { 
			  markReadNotification ： { 
			    notification ： { 
			      seenState ： SEEN ，
			    } ，
			  } ，
			} ; 
			
			commitMutation （ 
			  environment ，
			  { 
			    mutation ， 
			    optimisticResponse ， 
			    variables ，
			  } ，
			）;

	**Config:**

	1. NODE_DELETE(响应被删除的数据节点，配置deletedIDFieldName: string)
		
			const mutation = graphql`
				  mutation DestroyShipMutation($input: DestroyShipData!) {
				    destroyShip(input: $input) {
				      destroyedShipId
				      faction {
				        ships {
				          id
				        }
				      }
				    }
				  }
				`;
				
				const configs = [{
				  type: 'NODE_DELETE',
				  deletedIDFieldName: 'destroyedShipId',
				}];

	

	1. RANGE_ADD (给定父节点，有关连接的信息以及响应负载中的新创建边的名称。Relay将根据connectionInfo中指定的范围行为将节点添加到存储并将其附加到连接。)
	
 	 	**参数**：
		
		- parentID: string：包含连接的父节点的DataID。
		
		- connectionInfo: Array<{key: string, filters?: Variables, rangeBehavior: string}>：一个包含连接键的对象数组，一个包含可选过滤器的对象，以及一个取决于我们期望的行为（追加，前置或忽略）的范围行为。
		
		- filters：包含GraphQL调用的对象，例如const filters = {'orderby': 'chronological'};。
		
		- edgeName: string：表示新创建的边的响应中的字段名称

		**Example**:

			const mutation = graphql`
			  mutation AddShipMutation($input: AddShipData!) {
			    addShip(input: $input) {
			      faction {
			        ships {
			          id
			        }
			      }
			      newShipEdge
			    }
			  }
			`;
			
			const configs = [{
			  type: 'RANGE_ADD',
			  parentID: 'shipId',
			  connectionInfo: [{
			    key: 'AddShip_ships',
			    rangeBehavior: 'append',
			  }],
			  edgeName: 'newShipEdge',
			}];


	1. RANGE_DELETE(给定父节点，连接键，响应负载中的一个或多个DataID以及父节点和连接之间的路径，Relay将从连接中删除节点，但将关联的记录留在存储区中。) 

		**参数：**

		- parentID: string：包含连接的父节点的DataID
		
		- connectionKeys: Array<{key: string, filters?: Variables}>：包含连接键和可选过滤器的对象数组。
		
		- filters：包含GraphQL调用的对象，例如const filters = {'orderby': 'chronological'};
		
		- pathToConnection: Array<string>：包含父级和连接之间的字段名称的数组，包括父级和连接。
		
		- deletedIDFieldName: string | Array<string>：包含已删除节点的DataID的响应中的字段名称，或从连接中删除的节点的路径

		**Example**

			const mutation = graphql`
			  mutation RemoveTagsMutation($input: RemoveTagsData!) {
			    removeTags(input: $input) {
			      todo {
			        tags {
			          id
			        }
			      }
			      removedTagId
			    }
			  }
			`;
			
			const configs = [{
			  type: 'RANGE_DELETE',
			  parentID: 'todoId',
			  connectionKeys: [{
			    key: RemoveTags_tags,
			    rangeBehavior: 'append',
			  }],
			  pathToConnection: ['todo', 'tags'],
			  deletedIDFieldName: removedTagId
			}];



1. Subscriptions

	**API如下：**

		const {requestSubscription} = require('react-relay');
		
		type Variables = {[name: string]: any};
		
		requestSubscription(
		  environment: Environment,
		  config: {
		    subscription: GraphQLTaggedNode,
		    variables: Variables,
		    onCompleted?: ?() => void,
		    onError?: ?(error: Error) => void,
		    onNext?: ?(response: ?Object) => void,
		    updater?: ?(store: RecordSourceSelectorProxy, data: SelectorData) => void,
		    configs?: Array<RelayMutationConfig>,
		  },
		);

	**Example如下，以下只用到subscription  和	variables,以下是一个简单的订阅，当你只是更改记录的属性时，可以通过以下识别ID：**

		const {
		  requestSubscription,
		  graphql,
		} = require('react-relay');
		
		const subscription = graphql`
		  subscription MarkReadNotificationSubscription(
		    $storyID: ID!
		  ) {
		    markReadNotification(storyID: $storyID) {
		      notification {
		        seenState
		      }
		    }
		  }
		`;
		
		const variables = {
		  storyID,
		};
		
		requestSubscription(
		  yourEnvironment, // see Environment docs
		  {
		    subscription,
		    variables,
		    // optional but recommended:
		    onCompleted: () => {/* server closed the subscription */},
		    onError: error => console.error(error),
		  }
		);

	**在响应中更新客户端**
		
	当收到订阅响应时，您可能希望执行自定义逻辑来更新Relay的内存缓存。为此，传递一个updater函数：

		const {ConnectionHandler} = require('relay-runtime');
		
		requestSubscription(
		  environment,
		  {
		    subscription,
		    variables,
		    updater: store => {
		      // Get the notification
		      const rootField = store.getRootField('markReadNotification');
		      const notification = rootField.getLinkedRecord('notification');
		      // Add it to a connection
		      const viewer = store.getRoot().getLinkedRecord('viewer');
		      const notifications =
		        ConnectionHandler.getConnection(viewer, 'notifications');
		      const edge = ConnectionHandler.createEdge(
		        store,
		        notifications,
		        notification,
		        '<TypeOfNotificationsEdge>',
		      );
		      ConnectionHandler.insertEdgeAfter(notifications, edge);
		    },
		  },
		);


1. babel-plugin-relay

	**Relay用于转换Graphql语句为requires，如下：**
		
		graphql`
		  fragment MyComponent on Type {
		    field
		  }
		`
	转换为：

		function () {
		  return require('./__generated__/MyComponent.graphql');
		}

	使用yarn安装：

		yarn add --dev babel-plugin-relay

	然后将其添加到`.babelrc`文件中

		{
		  "plugins": [
		    "relay"
		  ]
		}


1. Relay Compiler

	Relay Moderny使用Relay Compiler转换Graphql语句（同上）为你文件中的_generated文件夹内的文件。以上的片段将转换后的文件出现在`./__generated__/MyComponent.graphql`,在运行时可以静态使用。通过提前构建查询，客户端的JS运行时不负责生成查询字符串，并且可以在构建步骤中合并查询中重复的字段，以提高解析效率。

	**设置Relay Compiler**

	首先你要安装watchman
	
		yarn add watchman
	
	然后安装relay-compiler

		yarn add --dev relay-compiler

	在`package.json`文件中增加一句script:

		"scripts": {
			  "relay": "relay-compiler --src ./src --schema ./schema.graphql"
			}
	>注意编辑完文件后运行relay命令，使其重新生成新的文件

	或者可以选择全局安装

		yarn global add relay-compiler
	
	在对应用程序文件进行编辑之后，运行`relay-compiler --src ./src --schema path/schema.graphql`以生成新文件，或者`relay-compiler --src ./src --schema path/schema.graphql --watch `将编译器作为长期运行的进程运行，每当您保存时都会自动生成新文件。

	**使用Relay Compiler，你需要一个`.graphql`或者`.json`文件，用于描述GraphQL server API,这个文件为本地服务器资源的表示，不能直接编辑，例如你有如下的一个`schema.graphql`:**

		schema {
		  query: Root
		}
		
		type Root {
		  dictionary: [Word]
		}
		
		type Word {
		  id: String!
		  definition: WordDefinition
		}
		
		type WordDefinition {
		  text: String
		  image: String
		}





		








 
		  

