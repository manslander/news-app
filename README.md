import 'package:flutter/material.dart';
import 'package:dio/dio.dart';

void main() {
  runApp(NewsApp());
}

class NewsApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'News App',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: NewsListPage(),
    );
  }
}

class NewsListPage extends StatefulWidget {
  const NewsListPage({super.key});

  @override
  _NewsListPageState createState() => _NewsListPageState();
}

class _NewsListPageState extends State<NewsListPage> {
  List<dynamic> _articles = [];
  bool _isLoading = true;
  String _errorMessage = '';
  final Dio _dio = Dio();

  @override
  void initState() {
    super.initState();
    _fetchNews();
  }

  Future<void> _fetchNews() async {
    setState(() {
      _isLoading = true;
      _errorMessage = '';
    });
    try {
      final response = await _dio.get('4132fa3406084e568598f992ea209a7f');
      if (response.statusCode == 200) {
        setState(() {
          _articles = response.data['articles'];
          _isLoading = false;
        });
      } else {
        setState(() {
          _errorMessage =
              'Failed to load news. Status code: ${response.statusCode}';
          _isLoading = false;
        });
      }
    } catch (e) {
      setState(() {
        _errorMessage = 'An error occurred: $e';
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Top Headlines')),
      body:
          _isLoading
              ? Center(child: CircularProgressIndicator())
              : _errorMessage.isNotEmpty
              ? Center(child: Text(_errorMessage))
              : ListView.builder(
                itemCount: _articles.length,
                itemBuilder: (context, index) {
                  final article = _articles[index];
                  return Card(
                    child: ListTile(
                      leading:
                          article['urlToImage'] != null
                              ? Image.network(
                                article['urlToImage'],
                                width: 50,
                                height: 50,
                                fit: BoxFit.cover,
                                errorBuilder:
                                    (context, error, stackTrace) =>
                                        Icon(Icons.image_not_supported),
                              )
                              : Icon(Icons.image_not_supported),
                      title: Text(article['title'] ?? 'No Title'),
                    ),
                  );
                },
              ),
    );
  }
}
